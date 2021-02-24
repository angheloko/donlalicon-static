---
title: "Automatically including all relationships in Drupal JSON:API"
createdAt: 2021-02-24
tags:
  - Drupal
  - "JSON:API"
cover:
  image: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/karine-avetisyan-ipuiM-36tAg-unsplash.jpg?alt=media&token=67bb6ded-a860-4160-9263-da8b45b611a6
  thumb: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/karine-avetisyan-ipuiM-36tAg-unsplash-thumb.jpg?alt=media&token=fe095591-ca03-4692-9171-d22b0e8e49f7
  caption: Photo by <a href="https://unsplash.com/@kar111?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Karine Avetisyan</a> on <a href="https://unsplash.com/s/photos/chain?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a>
---

There is currently no way to automatically include all relationships when fetching resources and this is actually intentional and by design.

It should be up to the client to explicitly define any additional data that it requires. However, there are scenarios where all data is *really* required and having the client manage this list of data to be included can be tedious and messy as new relationships are added.

## Reference fields

There are certain field types that allow you to reference other entities (e.g. node, paragraph, etc), and these are the fields that creates relationships. The most common of these field types are:

- Entity reference field
- Entity reference revision field
- Image field

We can then write code that checks if an entity's field is one that references another entity and automatically include the JSON:API of the entity that is being referenced.

## Custom service

We create a [custom service](https://www.drupal.org/docs/drupal-apis/services-and-dependency-injection/services-and-dependency-injection-in-drupal-8#s-defining-your-own-services) to make the code reusable. The code below recursively retrieves all relationships base on the type of the field and the type of the entity it references.

### Dependencies

To serialize an entity object into JSON:API format, we rely on a [helper function](https://git.drupalcode.org/project/jsonapi_extras/-/blob/8.x-3.x/src/EntityToJsonApi.php#L89) provided by the [`jsonapi_extras`](https://www.drupal.org/project/jsonapi_extras) module. 

```php[my_module/src/JsonApiBuilder]
<?php

namespace Drupal\my_module;

use Drupal\Core\Entity\EntityFieldManagerInterface;
use Drupal\Core\Entity\EntityInterface;
use Drupal\jsonapi_extras\EntityToJsonApi;

/**
 * JsonApiBuilder service.
 */
class JsonApiBuilder {

  /**
   * The EntityToJsonApi service.
   *
   * @var \Drupal\jsonapi_extras\EntityToJsonApi
   */
  protected $entityToJsonApi;

  /**
   * The EntityFieldManager.
   *
   * @var \Drupal\Core\Entity\EntityFieldManagerInterface
   */
  protected $entityFieldManager;

  /**
   * Constructs a JsonApiBuilder object.
   *
   * @param \Drupal\jsonapi_extras\EntityToJsonApi $entity_to_jsonapi
   * @param \Drupal\Core\Entity\EntityFieldManagerInterface $entity_field_manager
   */
  public function __construct(EntityToJsonApi $entity_to_jsonapi, EntityFieldManagerInterface $entity_field_manager) {
    $this->entityToJsonApi = $entity_to_jsonapi;
    $this->entityFieldManager = $entity_field_manager;
  }

  /**
   * Method description.
   *
   * @param \Drupal\Core\Entity\EntityInterface $entity
   *
   * @return string
   * @throws \Exception
   */
  public function buildJsonApi(EntityInterface $entity): string {
    $includes = $this->getIncludes($entity);
    return $this->entityToJsonApi->serialize($entity, $includes);
  }

  /**
   * Recursively retrieve an array of field paths that's suitable for the
   * include parameter of a JSON:API request.
   *
   * @param \Drupal\Core\Entity\EntityInterface $entity
   *
   * @return array
   */
  public function getIncludes(EntityInterface $entity): array {
    // Field types that can reference other entities.
    $field_types = [
      'entity_reference_revisions',
      'entity_reference',
      'image',
    ];

    // Referencable entities that we want to include.
    $target_types = [
      'paragraph',
      'media',
      'file',
      'node',
    ];

    $includes = [];

    // Iterate the fields of an entity and look for fields that can reference
    // other entities.
    $field_definitions = $this->entityFieldManager->getFieldDefinitions($entity->getEntityTypeId(), $entity->bundle());
    foreach ($field_definitions as $field_definition) {
      if (!$field_definition->getFieldStorageDefinition()->isBaseField()) {
        // Only include specific field types.
        if (in_array($field_definition->getType(), $field_types)) {
          // Only include specific entities.
          if (in_array($field_definition->getSetting('target_type'), $target_types)) {
            $includes[] = $field_definition->getName();

            // Get the referenced entities and also get their relationships.
            $referenced_entities = $entity->get($field_definition->getName())->referencedEntities();
            foreach ($referenced_entities as $referenced_entity) {
              $_includes = $this->getIncludes($referenced_entity);
              foreach ($_includes as $_include) {
                $full_path = $field_definition->getName() . '.' . $_include;
                if (!in_array($full_path, $includes)) {
                  $includes[] = $full_path;
                }
              }
            }
          }
        }
      }
    }

    return $includes;
  }

}
```

Don't forget to define the service in the `services.yml` file:

```yml[my_module/my_module.services.yml]
services:
  my_module.jsonapi_builder:
    class: Drupal\my_module\JsonApiBuilder
    arguments: ['@jsonapi_extras.entity.to_jsonapi', '@entity_field.manager']
```

## Usage

Now that we have service, accessing it can be done via dependency injection or via the generic `\Drupal::service()` method.

Example:

```php
$jsonapi_builder = \Drupal::service('my_module.jsonapi_builder');
$output = $jsonapi_builder->buildJsonApi($entity);
```
