---
title: "Adding Structured Data to Your Nuxt Pages"
subtitle: "Helping Google Search by adding structured data into your Nuxt pages"
lead: "Helping Google Search by adding structured data into your Nuxt pages"
description: "Helping Google Search by adding structured data into your Nuxt pages"
createdAt: 2020-01-12T03:28:34.215Z
updatedAt: 2020-01-12T04:08:14.685Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/adding-structured-data-your-nuxt-pages%2Fsai-kiran-anagani-5Ntkpxqt54Y-unsplash.jpg?alt=media&token=7e7f080b-fabd-4b42-a3db-c5d93735ddb6"
  alt: "Adding Structured Data to Your Nuxt Pages cover image"
  caption: "Photo by Sai Kiran Anagani on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/adding-structured-data-your-nuxt-pages%2Fsai-kiran-anagani-5Ntkpxqt54Y-unsplash_200x200.jpg?alt=media&token=7e7f080b-fabd-4b42-a3db-c5d93735ddb6"
tags: 
  - Structured data
  - Google Search
  - Nuxt
  - Vue Meta
---
[Structured data](https://developers.google.com/search/docs/guides/intro-structured-data) helps Google Search understand the content of your pages better. It's a standardised format for providing information about a page and classifying the page content.

You can add structured data by using the [JSON-LD](https://json-ld.org/) format. Below is an example of a structured data for a simple article:

```html
<script type="application/ld+json">
{
  "@type": "Article",
  "headline": "Article headline",
  "image": "https://example.com/photos/1x1/photo.jpg",
  "datePublished": "2015-02-05T08:00:00+08:00",
  "dateModified": "2015-02-05T09:20:00+08:00"
}
</script>
```

There are other [properties](https://developers.google.com/search/docs/data-types/article#amp) that you can specify in the structured data depending on the type of content. For instance, a recipe content type could have `cookTime` and `recipeInstructions`.

Adding structured data via the head method
------------------------------------------

I decided to add the structure data inside the `<head>` section of the page so naturally I looked into the [head method](https://nuxtjs.org/api/pages-head#the-head-method), which uses [vue-meta](https://github.com/nuxt/vue-meta).

Since I was adding a script tag, I ended up with the following (trimmed for brevity):

```html
<script>
export default {
  head () {
    const dateCreated = new Date(this.blog.created.seconds * 1000)
    const dateChanged = new Date(this.blog.changed.seconds * 1000)

    const structuredData = {
      '@type': 'Article',
      datePublished: dateCreated.toISOString(),
      dateModified: dateChanged.toISOString(),
      headline: this.blog.title,
      image: this.blog.imageUrl
    }

    return {
      script: [
        {
          type: 'application/ld+json',
          json: structuredData
        }
      ]
    }
  }
}
</script>
```

First, we construct the structured data from the blog data and add it into the [script property](https://vue-meta.nuxtjs.org/api/#script). Luckily, Vue Meta supports adding [JSON objects into script tags](https://vue-meta.nuxtjs.org/api/#add-json-data) since version 2.1.

You can see the full [source code](https://github.com/angheloko/donlalicon/blob/master/pages/blog/_id/index.vue) from the repo.

Pre-Vue Meta 2.1
----------------

If for some reason you are stuck with an older version of Vue Meta, you can still inject custom code into the script tag inside the head of your page with the use of `__dangerouslyDisableSanitizers` or `__dangerouslyDisableSanitizersByTagID`.

Example of how the earlier code will be adjusted:

```js
return {
  __dangerouslyDisableSanitizers: ['script'],
  script: [
    {
      type: 'application/ld+json',
      innerHTML: JSON.stringify(structuredData)
    }
  ]
}
```

Base from the name of these properties, it goes without saying that their use is **highly discouraged**.

Closing
-------

You can add structured data in [other formats](https://developers.google.com/search/docs/guides/intro-structured-data#structured-data-format) too, although, Google recommends `JSON+LD`.

You can also add the structured data in the page's body instead of head. You can even inject it dynamically, according to the documentation. My tests say otherwise but your results may vary.

I opted to initialise the blog data in the `asyncData` method and insert the structured data into the head. Afterall, doing both is straightforward with Nuxt and Vue Meta.