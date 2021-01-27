---
title: "Essential Modules for a Decoupled Drupal"
subtitle: "Where to start in your path to decoupling Drupal"
lead: "Where to start in your path to decoupling Drupal"
description: ""
createdAt: 2020-03-05T08:20:35.770Z
updatedAt: 2020-03-27T01:52:02.120Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fwill-francis-Rm3nWQiDTzg-unsplash.jpg?alt=media&token=ac4464ef-053b-4df9-a4e0-13bc60190c19"
  alt: "Essential Modules for a Decoupled Drupal cover image"
  caption: "Photo by Will Francis on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fwill-francis-Rm3nWQiDTzg-unsplash_thumb.jpg?alt=media&token=aaa9243a-94eb-44ea-81eb-8facdc47f56e"
tags: 
  - Drupal
  - Decoupled
  - JSONAPI
  - Headless CMS
---
There are many reasons to decouple a Drupal website and as new front-end frameworks and libraries such as Vue.js and React become increasingly popular, the idea of decoupling a Drupal website becomes more appealing. Besides obtaining front-end freedom, reasons related to scalability, performance, and reusability make this move more compelling. Here are some ways on how to get you started on the decoupled path.

Essential modules
-----------------

### JSONAPI

The [JSON:API](https://www.drupal.org/docs/8/core/modules/jsonapi-module) module has been part of core since 8.7. This module exposes REST APIs for every entity type in your Drupal application using the [JSON:API specification](https://jsonapi.org/). As someone who has developed custom web APIs before for Drupal applications, using this module prevents a lot of bike-shedding while offering almost everything you need out of the box.

### Decoupled router

One of the biggest nuances in a decoupled architecture is route management. In my opinion, this stems from the fact that route management is easily handled by the CMS but as soon as we decouple content management and content presentation, issues like this arise.

The [Decoupled router](https://www.drupal.org/project/decoupled_router) module tackles this issue by resolving paths and locating the content that the path eventually leads to. This module works very well even with aliases and redirects.

### Subrequests

Decreasing the number of requests you send to the server is one of the things you can do to optimize your web application and the module [Subrequests](https://www.drupal.org/project/subrequests) lets your Drupal-backed web application handle multiple requests contained in a single request.

### Other modules

Some modules that can also contribute to and enhance your decoupled Drupal are:

**[Simple OAuth](https://www.drupal.org/project/simple_oauth)**

Authenticates incoming requests using OAuth 2.0.

**[Consumers](https://www.drupal.org/project/consumers)**

With this module, you can identify and provide certain features to a specific application that connects to your decoupled Drupal.

**[Paragraphs](https://www.drupal.org/project/paragraphs)**

Let's you define and customize complex nested data structures. In a component-based web application, this module will surely come in handy.

Distributions
-------------

If you need a distribution that's built and optimized for a decoupled architecture, you can choose between [Contenta CMS](https://www.contentacms.org/) and [Headless Lightning](https://github.com/acquia/headless_lightning).