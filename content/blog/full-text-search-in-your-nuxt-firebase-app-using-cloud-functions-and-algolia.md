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
<p>Cloud Firestore doesn't provide a way to index and search text fields. It doesn't provide an easy way to perform pattern matching either. As most applications grow, so does its content and allowing users to search for content easily becomes more important.</p><p>According to the <a href="https://firebase.google.com/docs/firestore/solutions/search" rel="noopener noreferrer nofollow">Firebase documentation</a>, we can consider using third-party search services like <a href="https://www.algolia.com/" rel="noopener noreferrer nofollow">Algolia</a> to provide our search features.</p><h2>Indexing with Cloud Functions</h2><p><a href="https://firebase.google.com/docs/functions" rel="noopener noreferrer nofollow">Cloud Functions</a> are basically code that runs in response to certain events like when a user logs in or when new content is created.</p><p>Since we want to index our content, the events that we're interested are those triggered when a new content is created, updated, and/or deleted. For this we can use the <a href="https://firebase.google.com/docs/reference/functions/providers_firestore_.documentbuilder.html#on-write" rel="noopener noreferrer nofollow"><strong>onWrite</strong></a> event. The code below is <a href="https://github.com/angheloko/donlalicon/blob/master/functions/index.js#L8" rel="noopener noreferrer nofollow">based on how this blog indexes its content using Cloud Functions</a>.</p><pre><code>exports.indexBlog = functions.firestore.document('blogs/{blogId}').onWrite((change, context) =&gt; {
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
      .then(() =&gt; {
        return true
      })
      .catch((error) =&gt; {
        console.error('Error deleting blog from index', error)
      })
  }

  function saveObject() {
    // The body property is stripped of HTMl tags and stop words.
    return index
      .saveObject({
        objectID: blogId,
        title: document.title,
        body: stopword.removeStopwords(document.body.replace(/(&lt;([^&gt;]+)&gt;)/ig,"").split(' ')).join(' ').replace(/\s\s+/g, ' '),
        tags: document.tags,
        changed: document.changed.toMillis()
      })
      .then(() =&gt; {
        return true
      })
      .catch((error) =&gt; {
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
})</code></pre><h2>The Search Box</h2><p>Now that we index our content, all that's left is to provide a search box in our app. We can create a custom Vue component and use Algolia's <a href="https://www.algolia.com/doc/api-client/getting-started/what-is-the-api-client/javascript/?language=javascript" rel="noopener noreferrer nofollow">Javascript API client</a>, or we can use the <a href="https://www.algolia.com/doc/guides/building-search-ui/what-is-instantsearch/vue/" rel="noopener noreferrer nofollow">official Vue components library</a>.</p><p>For this blog, I went with option 2 since using the Vue library not only handles the actual integration with Algolia's Search API but also provides flexible components like the search autocomplete field.</p><p>Once you've installed the <a href="https://www.algolia.com/doc/guides/building-search-ui/installation/vue/" rel="noopener noreferrer nofollow">Vue library</a>, we'll need to create a <a href="https://nuxtjs.org/guide/plugins/" rel="noopener noreferrer nofollow">Nuxt plugin</a> for it:</p><p><code>plugins/vue-instantsearch.js</code></p><pre><code>import Vue from 'vue'
import InstantSearch from 'vue-instantsearch'

Vue.use(InstantSearch)</code></pre><p>We can then start using the plugin, along with all of Algolia's <a href="https://www.algolia.com/doc/api-reference/widgets/vue/" rel="noopener noreferrer nofollow">Vue components</a>. In the code below, the <a href="https://www.algolia.com/doc/api-reference/widgets/instantsearch/vue/" rel="noopener noreferrer nofollow">ais-instant-search</a>, <a href="https://www.algolia.com/doc/api-reference/widgets/autocomplete/vue/" rel="noopener noreferrer nofollow">ais-autocomplete</a>, and <a href="https://www.algolia.com/doc/api-reference/widgets/highlight/vue/" rel="noopener noreferrer nofollow">ais-highlight</a> are all components from library.</p><pre><code>&lt;template&gt;
  &lt;client-only&gt;
    &lt;ais-instant-search
        v-click-outside="closeAutocomplete"
        :search-client="searchClient"
        index-name="blogs"
    &gt;
      &lt;ais-autocomplete&gt;
        &lt;div slot-scope="{ indices, refine }"&gt;
          &lt;div&gt;
            &lt;input
              v-model="keywords"
              type="search"
              placeholder="Search"
              @input="refine($event.currentTarget.value)"
            &gt;
            &lt;div v-if="keywords &amp;&amp; !hideAutocomplete"&gt;
              &lt;a 
                v-for="hit in indices[0].hits"
                :key="hit.objectID"
                @click="goToBlog(hit.objectID)"
              &gt;
                &lt;ais-highlight attribute="title" :hit="hit" /&gt;
              &lt;/a&gt;
            &lt;/div&gt;
          &lt;/div&gt;
        &lt;/div&gt;
      &lt;/ais-autocomplete&gt;
    &lt;/ais-instant-search&gt;
  &lt;/client-only&gt;
&lt;/template&gt;
&lt;script&gt;
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
&lt;/script&gt;</code></pre><p>The code above is base from <a href="https://github.com/angheloko/donlalicon/blob/master/layouts/default.vue" rel="noopener noreferrer nofollow">this blog's code</a> and is edited for brevity.</p><h2>Closing</h2><p>While Cloud Firestore doesn't offer an out-of-the-box feature for indexing and searching, third-party services exist that are perfect for this job and integrating these third-party services into your app have never been easier.</p><p></p><p></p>