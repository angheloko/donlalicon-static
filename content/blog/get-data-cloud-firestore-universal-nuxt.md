---
title: "Getting Data From Cloud Firestore Into Your Universal Nuxt App"
subtitle: "How to retrieve data from Cloud Firestore and use it in your Universal Nuxt app"
lead: "How to retrieve data from Cloud Firestore and use it in your Universal Nuxt app"
description: "How to retrieve data from Cloud Firestore and use it in out Universal Nuxt app"
createdAt: 2020-01-04T02:23:16.121Z
updatedAt: 2020-01-05T02:17:09.439Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/get-data-cloud-firestore-universal-nuxt%2Fmaxresdefault.jpg?alt=media&token=b479d10d-e23c-4f2f-a709-23b9fdad3de7"
  alt: "Getting Data From Cloud Firestore Into Your Universal Nuxt App cover image"
  caption: ""
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/get-data-cloud-firestore-universal-nuxt%2Ffirestore-logo.png?alt=media&token=c5c15d87-8442-4d9c-aa7b-b07c54708c40"
tags: 
  - Firestore
  - Nuxt
  - SSR
---
<p>In the <a href="https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase" rel="noopener noreferrer nofollow">previous article</a>, we created a <a href="https://nuxtjs.org/guide/plugins" rel="noopener noreferrer nofollow">plugin</a> to integrate our Nuxt app with Firebase. By doing so, we can now easily access Firebase's features anywhere in the app.</p><p>In this article, we will retrieve data from Firestore from 2 commonly used methods:</p><ol><li><p>Within any Vue instance lifecycle (e.g. <code>mounted</code>)</p></li><li><p><code>asyncData</code></p></li></ol><p>And we'll also be using 2 different styles:</p><ol><li><p>Using promises</p></li><li><p>Using <code>async/await</code>.</p></li></ol><h2>Prerequisites</h2><ul><li><p><a href="https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase" rel="noopener noreferrer nofollow">Create a Nuxt plugin</a> to integrate our app with Firebase.</p></li></ul><h2>Within any Vue instance lifecycle</h2><p>In our previous article, we injected the Firebase object into the Vue instance. This allows us to access the Firebase object via <code>this</code> anywhere in the instance's lifecycle, such as in the <code>mounted</code> method or in any custom methods defined in the <code>methods</code> property.</p><h3>Using promises</h3><pre><code>export default {
  data () {
    return {
      blogs: []
    }
  },
  mounted () {
    const db = this.$firebase.firestore()

    db.collection('blogs')
      .orderBy('created', 'desc')
      .get()
      .then((querySnapshot) =&gt; {
        querySnapshot.forEach((doc) =&gt; {
          this.blogs.push({
            id: doc.id,
            ...doc.data()
          })
        })
      })
  }
}</code></pre><h3>Using async/await</h3><pre><code>export default {
  data () {
    return {
      blogs: []
    }
  },
  async mounted () {
    const db = this.$firebase.firestore()

    const querySnapshot = await db.collection('blogs')
      .orderBy('created', 'desc')
      .get()

    querySnapshot.forEach((doc) =&gt; {
      this.blogs.push({
        id: doc.id,
        ...doc.data()
      })
    })
  }
}</code></pre><h2>Using the asyncData method</h2><p>Since the <code>asyncData</code> method runs before the Vue instance is initialised, we don't have access to <code>this</code> yet. However, in our <a href="https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase" rel="noopener noreferrer nofollow">previous article</a>, we also injected the Firebase object into the <code>context</code> object. This allows us to access the Firebase object via <a href="https://nuxtjs.org/api/context#the-context" rel="noopener noreferrer nofollow">context</a> parameter.</p><p>I love using this method because it allows us to retrieve data and process it on the server. That means we can pre-render the content of the page making it immediately available to search engine crawlers.</p><p>The value returned by this method will be merged with any data defined in the <code>data</code> method.</p><h3>Returning a promise</h3><pre><code>asyncData ({ app }) {
  const teasers = []
  const db = app.$firebase.firestore()

  return db.collection('teasers')
    .where('published', '==', true)
    .orderBy('created', 'desc')
    .get()
    .then((querySnapshot) =&gt; {
      for (const doc of querySnapshot.docs) {
        teasers.push({
          id: doc.id,
          ...doc.data()
        })
      }
      return { teasers }
    })
}</code></pre><h3>Using async/await</h3><pre><code> async asyncData ({ app }) {
  const teasers = []
  const db = app.$firebase.firestore()

  const querySnapshot = await db.collection('teasers')
    .where('published', '==', true)
    .orderBy('created', 'desc')
    .get()

  if (querySnapshot.size &gt; 0) {
    for (const doc of querySnapshot.docs) {
      teasers.push({
        id: doc.id,
        ...doc.data()
      })
    }
  }

  return { teasers }
}</code></pre><h3>Why pre-render data</h3><p>We need to pre-render data on the server whenever we need information to be present immediately inside our initial HTML response. This is useful for making our pages SEO-friendly since we can the content is present in the initial HTML response. In traditional single-page applications, the content of the page usually comes after the initial HTMl is rendered.</p><h3>Setting header data and important content on the server</h3><pre><code>&lt;template&gt;
  &lt;div&gt;{{ blog.body }}&lt;/div&gt;
&lt;/template&gt;
&lt;script&gt;
export default {
  data () {
    return {
      blog: null
    }
  },
  head () {
    // this.blog is pre-populated in asyncData().
    const head = {
      title: this.blog.title,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.blog.description
        }
      ]
    }
    return head
  },
  async asyncData ({ app, params, error }) {
    const db = app.$firebase.firestore()

    try {
      const documentSnapshot = await db.collection('blogs').doc(params.id).get()

      if (!documentSnapshot.exists) {
        error({ statusCode: 404, message: 'Blog not found' })
        return
      }

      // Returned value is merged with the values defined in data().
      return {
        blog: {
          id: documentSnapshot.id,
          ...documentSnapshot.data()
        }
      }
    } catch (e) {
      error({ statusCode: 404, message: 'Blog not found' })
    }
  }
}
&lt;/script&gt;</code></pre><p>The code above does the following on the server:</p><ol><li><p>Retrieve data using the <code>asyncData</code>.</p></li><li><p>Use the data in the <code>&lt;head&gt;</code> section of the page.</p></li><li><p>Use the data in the content.</p></li></ol><h2>Conclusion</h2><p>Using Firestore seemed like a breeze to me. It is a convenient, low-cost, low-maintenance data storage solution. Its documentation is clear and abundant, and its community vibrant. </p><p>Being able to access to the Firebase object from anywhere in the code makes coding a lot easier. I also feel that using <code>async/await</code> made the code cleaner and easier to understand.</p><p>Finally, if we need content to be pre-rendered, we can use <code>asyncData</code>. This is especially useful for making our apps SEO-friendly.</p>