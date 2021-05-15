---
title: Using Nuxt Content as a database
description: Using Nuxt Content as a database
cover:
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/nuxt-content.png?alt=media&token=1a1cf59d-2405-417a-a849-63a28061206b"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/nuxt-content.png?alt=media&token=1a1cf59d-2405-417a-a849-63a28061206b"
tags:
  - NuxtJS
  - Nuxt Content
---
In the [previous article](/blog/monster-hunter-rise-builder), I shared a [new simple tool](https://mhr-builder.com) that lets Monster Hunter Rise players plan, build, and compare sets. In this article, I'll share how I'm managing the data for that app using [Nuxt Content](https://content.nuxtjs.org).

## An inexpensive and efficient database

Nuxt Content is an inexpensive way of storing and managing structured data for your NuxtJS apps. It allows you to store your data in Markdown, JSON, YAML, XML and CSV formats, and best of all, it lets query these data easily using a MongoDB-like API.

## Data structure

Monster Hunter Rise has a wide range of information, from the plants that you can gather to the monsters that you can hunt. The tool that I developed focuses on combining the weapons and equipments that you can build base on your play style so that you can make your hunts easier and more fun.

All the data are stored under the [`/content`](https://github.com/angheloko/mhr-builder/tree/master/content) directory and are using the JSON format.

### Weapons

The weapons are categorized and stored base on their types (e.g. `bow-weapons.json`, `charge-blade-weapons.json`, etc.). An example of the structure is:

```json
{
  "name": "Kamura Iron Bow I",
  "slug": "kamura-iron-bow-i",
  "type": "bow",
  "slots": [],
  "attack": 60,
  "element": [],
  "affinity": 0,
  "defense": 0,
  "rarity": 1,
  "rampSkills": [
    "Attack Boost I",
    "Affinity Boost II",
    "Defense Boost I"
  ]
}
```

### Armor pieces

Armors are also categorized and stored base on their types (e.g. `arms-armors.json`, `chest-armors.json`, etc.). An example of the structure is:

```json
{
  "name": "Chainmail Gloves S",
  "type": "arms",
  "slug": "chainmail-gloves-s",
  "skills": [
    {
      "name": "Botanist",
      "slug": "botanist",
      "level": 1
    },
    {
      "name": "Defense Boost",
      "slug": "defense-boost",
      "level": 1
    }
  ],
  "slots": [
    1,
    1
  ],
  "rarity": 4,
  "baseDefense": 32,
  "fireResistance": 0,
  "waterResistance": 0,
  "thunderResistance": 1,
  "iceResistance": 0,
  "dragonResistance": 0
}
```

### Skills

All the skills are stored in `skills.json`. An example of the structure is:

```json
{
  "name": "Divine Blessing",
  "slug": "divine-blessing",
  "description": "Has a predetermined chance of reducing the damage you take.",
  "level": 3
}
```

### Decorations

All decorations are stored in `decorations.json`. An example of the structure is:

```json
{
  "name": "Absorber Jewel 1",
  "slug": "absorber-jewel-1",
  "skill": "Recoil Down",
  "skillSlug": "recoil-down",
  "level": 1
}
```

### Slugs

You may notice that every item, from weapons to decorations, have the `slug` field. This is important and without it, Nuxt Content will ignore the item. Think of the `slug` field as the item's unique ID.

## Fetching data

### Retrieving all data

Retrieving all data of a given resource (e.g. weapon, armor, skill, decoration) is very straightforward since it doesn't require any conditions. Example:

```js
this.$content(resourceName).fetch().then((results) => {
  // Do something with the results.
})
```

Where, `resourceName` would be the resource (or table) that you want to query. If I wanted to get all bows I'd pass `bow-weapons` since the filename where all the bows are stored is `bow-weapons.json`. If I wanted to get all head armor pieces I'd pass `head-armors` since the head armors are stored in `head-armors.json`.

### Filtering

An important feature for a set builder is being able to filter items base on a given criteria. For example, I wanted users to be able to filter armors base on a given skill it has. Nuxt Content uses [LokiJS](https://github.com/techfort/LokiJS) under the hood and as such it provides all, if not most, of the common querying methods that you'll need.

The code below is an excerpt of how I [query armors](https://github.com/angheloko/mhr-builder/blob/master/components/AddArmorModal.vue#L122) base on a given skill. In this snippet, we use the `$containsAny` operator since the `skills` property of the armor record is an array of objects.

```js[components/AddArmorModal.vue]
let queryBuilder = this.$content(this.resourceName)

const conditions = {}

conditions['skills.slug'] = {
  $containsAny: [value]
}

queryBuilder = queryBuilder.where(conditions)

queryBuilder.fetch().then((results) => {
  // Do something with the results.
})
```

You can also use the `$contains` operator if the compare value is **not** an array.

In addition, since the `skills` property of an armor is an object, I needed to enable the `nestedProperties` option of Nuxt Content. See [Nuxt Content options](#nuxt-content-options) on how I did it for this project.

This has got to be my most favorite feature of Nuxt Content. There are a number of operators that you can use such as `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, and more so be sure to check out the LokiJS's [documentation](https://github.com/techfort/LokiJS/wiki/Query-Examples#find-queries). 

### Full-text searching

Another useful feature of Nuxt Content is being able to do full-text search out of the box. In the builder app, users can search for any weapon, armor, or decoration by typing in some keywords. The excerpt below is used for [searching a decoration](https://github.com/angheloko/mhr-builder/blob/master/components/SetDecorationModal.vue#L75) using its name or skill:

```js[components/SetDecorationModal.vue]
let queryBuilder = this.$content('decorations')
  .where({
    level: this.level
  })
  .sortBy('name')

queryBuilder = queryBuilder.search(keywords)

queryBuilder.fetch().then((results) => {
  // Do something with the results.
})
```

The `search` method allows you to do full-text searching on a given field. You probably notice that I didn't explicitly specify the field and that's because I wanted to search using both the decoration's `name` and `skill` fields. 

The `search` method only allows searching with one field so if you want to be able to use multiple fields, you'll need specify the fields that you want to search on using the `fullTextSearchFields` options. See [Nuxt Content options](#nuxt-content-options) on how I did it for this project.

### Nuxt Content options

If you have a field that is an array or an object and you need Nuxt Content to be able to query with it, you'll need to set the `nestedProperties` option. See [Nuxt Content's nestedProperties option](https://content.nuxtjs.org/configuration/#nestedproperties) for more info.

If you have more than one fields that you want to be searchable via the `search` method, you'll need to set the `fullTextSearchFields` option. See [Nuxt Content's fulltextsearchfields option](https://content.nuxtjs.org/configuration/#fulltextsearchfields) for more info.

The excerpt below is based on my app's configuration:

```js[nuxt.config.js]
export default {
  content: {
    fullTextSearchFields: ['name', 'skill'],
    nestedProperties: ['skills.name', 'skills.slug', 'elements.type']
  }
}
```

## Closing

There are a number of tools that lets you manage data but if you want an inexpensive but powerful alternative for your NuxtJS project, I'd strongly recommend checking out [Nuxt Content](https://content.nuxtjs.org).

The [source code](https://github.com/angheloko/mhr-builder) for the app is open-source so check it out!
