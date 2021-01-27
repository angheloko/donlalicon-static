---
title: "Full-Text Search in Your Nuxt Firebase App Using Cloud Functions and Algolia"
subtitle: "Adding a search feature to your Firebase-backed Nuxt application using Algolia"
lead: "How to add a search feature to your Firebase-backed Nuxt application using Algolia"
description: ""
createdAt: 2020-03-14T05:11:47.696Z
updatedAt: 2020-03-27T01:47:30.726Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Falgolia-firestore(1).png?alt=media&token=f2b052ec-9524-4c76-b251-f40927869bf1"
  alt: "Full-Text Search in Your Nuxt Firebase App Using Cloud Functions and Algolia cover image"
  caption: ""
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Falgolia-firestore(1)_thumb.png?alt=media&token=54be2e5d-050e-4f95-97b8-eedd6a88a810"
tags: 
  - Nuxt
  - Algolia
  - Cloud Firestore
  - Cloud Functions
---
Cloud Firestore doesn't provide a way to index and search text fields. It doesn't provide an easy way to perform pattern matching either. As most applications grow, so does its content and allowing users to search for content easily becomes more important.

According to the [Firebase documentation](https://firebase.google.com/docs/firestore/solutions/search), we can consider using third-party search services like [Algolia](https://www.algolia.com/) to provide our search features.

Indexing with Cloud Functions
-----------------------------

[Cloud Functions](https://firebase.google.com/docs/functions) are basically code that runs in response to certain events like when a user logs in or when new content is created.

Since we want to index our content, the events that we're interested are those triggered when a new content is created, updated, and/or deleted. For this we can use the [**onWrite**](https://firebase.google.com/docs/reference/functions/providers_firestore_.documentbuilder.html#on-write) event. The code below is [based on how this blog indexes its content using Cloud Functions](https://github.com/angheloko/donlalicon/blob/master/functions/index.js#L8).

```js
exports.indexBlog = functions.firestore.document('blogs/{blogId}').onWrite((change, context) => {
  // "document" will be empty if it's deleted, otherwise, this contains
  // the updated values.
  const document = change.after.exists ? change.after.data() : null

  // Get the document ID. This will be used as the ID for the indexed content.
  const { blogId } = context.params

  // The API ID and key are stored using Cloud Functions config variables.
  // @see https://firebase.google.com/docs/functions/config-env
  const ALGOLIA_APP_ID= functions.config().algolia.app_id
  const ALGOLIA_API_KEY = functions.config().algolia.api_key
  
  // Create an Algolia Search API client.
  const client = algoliasearch(ALGOLIA_APP_ID, ALGOLIA_API_KEY)
  const index = client.initIndex('blogs')

  function deleteObject() {
    return index
      .deleteObject(blogId)
      .then(() => {
        return true
      })
      .catch((error) => {
        console.error('Error deleting blog from index', error)
      })
  }

  function saveObject() {
    // The body property is stripped of HTMl tags and stop words.
    return index
      .saveObject({
        objectID: blogId,
        title: document.title,
        body: stopword.removeStopwords(document.body.replace(/(<([^>]+)>)/ig,"").split(' ')).join(' ').replace(/\s\s+/g, ' '),
        tags: document.tags,
        changed: document.changed.toMillis()
      })
      .then(() => {
        return true
      })
      .catch((error) => {
        console.error('Error indexing blog', error)
      })
  }

  if (!document) {
    return deleteObject(blogId)
  } else {
    if (!document.published) {
      return deleteObject(blogId)
    } else {
      return saveObject()
    }
  }
})
```

The Search Box
--------------

Now that we index our content, all that's left is to provide a search box in our app. We can create a custom Vue component and use Algolia's [Javascript API client](https://www.algolia.com/doc/api-client/getting-started/what-is-the-api-client/javascript/?language=javascript), or we can use the [official Vue components library](https://www.algolia.com/doc/guides/building-search-ui/what-is-instantsearch/vue/).

For this blog, I went with option 2 since using the Vue library not only handles the actual integration with Algolia's Search API but also provides flexible components like the search autocomplete field.

Once you've installed the [Vue library](https://www.algolia.com/doc/guides/building-search-ui/installation/vue/), we'll need to create a [Nuxt plugin](https://nuxtjs.org/guide/plugins/) for it:

```js[plugins/vue-instantsearch.js]
import Vue from 'vue'
import InstantSearch from 'vue-instantsearch'

Vue.use(InstantSearch)
```

We can then start using the plugin, along with all of Algolia's [Vue components](https://www.algolia.com/doc/api-reference/widgets/vue/). In the code below, the [ais-instant-search](https://www.algolia.com/doc/api-reference/widgets/instantsearch/vue/), [ais-autocomplete](https://www.algolia.com/doc/api-reference/widgets/autocomplete/vue/), and [ais-highlight](https://www.algolia.com/doc/api-reference/widgets/highlight/vue/) are all components from library.

```vue
<template>
  <client-only>
    <ais-instant-search
        v-click-outside="closeAutocomplete"
        :search-client="searchClient"
        index-name="blogs"
    >
      <ais-autocomplete>
        <div slot-scope="{ indices, refine }">
          <div>
            <input
              v-model="keywords"
              type="search"
              placeholder="Search"
              @input="refine($event.currentTarget.value)"
            >
            <div v-if="keywords && !hideAutocomplete">
              <a 
                v-for="hit in indices[0].hits"
                :key="hit.objectID"
                @click="goToBlog(hit.objectID)"
              >
                <ais-highlight attribute="title" :hit="hit" />
              </a>
            </div>
          </div>
        </div>
      </ais-autocomplete>
    </ais-instant-search>
  </client-only>
</template>
<script>
import algoliasearch from 'algoliasearch/lite'

export default {
  data () {
    return {
      searchClient: algoliasearch(process.env.ALGOLIA_APPLICATION_ID, process.env.ALGOLIA_SEARCH_API_KEY),
      keywords: '',
      hideAutocomplete: false
    }
  },
  watch: {
    keywords (value) {
      if (value) {
        this.hideAutocomplete = false
      }
    }
  },
  methods: {
    goToBlog (blogId) {
      this.keywords = ''
      this.$router.push({
        name: 'blog-id',
        params: {
          id: blogId
        }
      })
    },
    closeAutocomplete () {
      this.hideAutocomplete = true
    }
  }
}
</script>
```

The code above is base from [this blog's code](https://github.com/angheloko/donlalicon/blob/master/layouts/default.vue) and is edited for brevity.

Closing
-------

While Cloud Firestore doesn't offer an out-of-the-box feature for indexing and searching, third-party services exist that are perfect for this job and integrating these third-party services into your app have never been easier.