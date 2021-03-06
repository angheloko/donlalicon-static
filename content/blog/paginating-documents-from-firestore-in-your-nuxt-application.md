---
title: "Paginating Documents From Firestore in Your Nuxt Application"
subtitle: "How to paginate documents in Firestore in your Nuxt application"
lead: "How to paginate documents in Firestore in your Nuxt application"
description: ""
createdAt: 2020-03-09T07:38:43.974Z
updatedAt: 2020-03-27T01:39:23.049Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fenrico-mantegazza-493arZWzpXM-unsplash.jpg?alt=media&token=32653709-c3d5-4bee-812d-59819ac684f2"
  alt: "Paginating Documents From Firestore in Your Nuxt Application cover image"
  caption: "Photo by Enrico Mantegazza on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fenrico-mantegazza-493arZWzpXM-unsplash_thumb.jpg?alt=media&token=25914b59-12dd-4719-95a2-e868c0073cab"
tags: 
  - Nuxt
  - Firestore
  - Pagination
---
Websites with a lot of content will offer a way to paginate it's content, either via infinite scroll, "Load more" button, or page numbers, and doing this with Firestore is pretty straightforward. Firestore uses [query cursors](https://firebase.google.com/docs/firestore/query-data/query-cursors) to define the starting and ending point of your query, which is very different from how one would do it with traditional relational databases where you use offsets and limits.

According to the documentation, you can use [scalar values](https://firebase.google.com/docs/firestore/query-data/query-cursors#add_a_simple_cursor_to_a_query) or a [document snapshot](https://firebase.google.com/docs/firestore/query-data/query-cursors#use_a_document_snapshot_to_define_the_query_cursor) as a cursor. I personally prefer using a document snapshot since I feel it's more precise and can avoid issues with documents that have fields with the same values.

For this article, we will use the [infinite scroll](https://www.smashingmagazine.com/2013/05/infinite-scrolling-lets-get-to-the-bottom-of-this/) method to page through our documents. This is the same technique that is used for [this website](https://github.com/angheloko/donlalicon/blob/master/pages/index.vue).

Infinite scrolling and loading
------------------------------

In the infinite scroll method, more content is automatically loaded as the user approaches the bottom of the page. So basically, we'll need to do the following:

1.  Detect if the user has scrolled to or is nearing the bottom of the page.
    
2.  If the user has reached or is near the end of the page, load the next batch of documents.
    
3.  Keep track of the last document that was loaded so we can use it as our starting point for the next batch of documents to load.
    
4.  If all documents have been loaded, do nothing.
    

The code below has been edited for brevity but you can view the [full code](https://github.com/angheloko/donlalicon/blob/master/pages/index.vue). Be sure to check the inline comments for more information.

```vue
<script>
export default {
  data () {
    return {
      teasers: [], // An array of documents in view.
      eof: false, // Flag to tell us if there's nothing left to load.
      isLoading: false, // Flag to tell us if we're currently in the process of loading the next batch.
      lastDoc: null, // The last loaded document snapshot.
      batchSize: 10 // The number of documents to load at a time.
    }
  },
  async mounted () {
    // Load the initial batch of documents.
    await this.loadBlogs()

    // Start listening to the scroll event.
    window.addEventListener('scroll', this.loadMore)
  },
  destroyed () {
    // Good practice to remove listeners when the component is destroyed.
    window.removeEventListener('scroll', this.loadMore)
  },
  methods: {
    async loadBlogs () {
      // Exit early if we are still loading or when there's nothing left to load.
      if (this.isLoading || this.eof) {
        return
      }

      this.isLoading = true
      const db = this.$firebase.firestore()

      let query = db.collection('teasers')
        .where('published', '==', true)
        .orderBy('created', 'desc')
        .limit(this.batchSize)

      // Start after where we ended.
      if (this.lastDoc) {
        query = query.startAfter(this.lastDoc)
      }

      const querySnapshot = await query.get()

      this.eof = querySnapshot.empty

      if (querySnapshot.size > 0) {
        // Keep track of the last loaded document.
        this.lastDoc = querySnapshot.docs[querySnapshot.docs.length - 1]

        for (const doc of querySnapshot.docs) {
          this.teasers.push({
            id: doc.id,
            ...doc.data()
          })
        }
      }

      this.isLoading = false
    },
    loadMore () {
      const elementBounds = this.$el.getBoundingClientRect()

      // Add extra padding to load earlier even before the bottom of the element is in view.
      const padding = 100

      const bottomOfWindow =
        elementBounds.bottom <=
        (window.innerHeight || document.documentElement.clientHeight) + padding

      if (bottomOfWindow && !this.isLoading && !this.eof) {
        this.loadBlogs()
      }
    }
  }
}
</script>
```

This code works well especially in a [page component](https://nuxtjs.org/guide/views#pages) since the [$el](https://012.vuejs.org/api/instance-properties.html#vm-\$el) property would refer the whole page.

You can see this code in action by visiting the [homepage](https://donlalicon.dev/) and you can view the [full code here](https://github.com/angheloko/donlalicon/blob/master/pages/index.vue).