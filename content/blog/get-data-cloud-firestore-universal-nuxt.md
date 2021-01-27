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
In the [previous article](https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase), we created a [plugin](https://nuxtjs.org/guide/plugins) to integrate our Nuxt app with Firebase. By doing so, we can now easily access Firebase's features anywhere in the app.

In this article, we will retrieve data from Firestore from 2 commonly used methods:

1.  Within any Vue instance lifecycle (e.g. `mounted`)
    
2.  `asyncData`
    

And we'll also be using 2 different styles:

1.  Using promises
    
2.  Using `async/await`.
    

Prerequisites
-------------

*   [Create a Nuxt plugin](https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase) to integrate our app with Firebase.
    

Within any Vue instance lifecycle
---------------------------------

In our previous article, we injected the Firebase object into the Vue instance. This allows us to access the Firebase object via `this` anywhere in the instance's lifecycle, such as in the `mounted` method or in any custom methods defined in the `methods` property.

### Using promises

```
export default {
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
      .then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
          this.blogs.push({
            id: doc.id,
            ...doc.data()
          })
        })
      })
  }
}
```

### Using async/await

```
export default {
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

    querySnapshot.forEach((doc) => {
      this.blogs.push({
        id: doc.id,
        ...doc.data()
      })
    })
  }
}
```

Using the asyncData method
--------------------------

Since the `asyncData` method runs before the Vue instance is initialised, we don't have access to `this` yet. However, in our [previous article](https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase), we also injected the Firebase object into the `context` object. This allows us to access the Firebase object via [context](https://nuxtjs.org/api/context#the-context) parameter.

I love using this method because it allows us to retrieve data and process it on the server. That means we can pre-render the content of the page making it immediately available to search engine crawlers.

The value returned by this method will be merged with any data defined in the `data` method.

### Returning a promise

```
asyncData ({ app }) {
  const teasers = []
  const db = app.$firebase.firestore()

  return db.collection('teasers')
    .where('published', '==', true)
    .orderBy('created', 'desc')
    .get()
    .then((querySnapshot) => {
      for (const doc of querySnapshot.docs) {
        teasers.push({
          id: doc.id,
          ...doc.data()
        })
      }
      return { teasers }
    })
}
```

### Using async/await

```
 async asyncData ({ app }) {
  const teasers = []
  const db = app.$firebase.firestore()

  const querySnapshot = await db.collection('teasers')
    .where('published', '==', true)
    .orderBy('created', 'desc')
    .get()

  if (querySnapshot.size > 0) {
    for (const doc of querySnapshot.docs) {
      teasers.push({
        id: doc.id,
        ...doc.data()
      })
    }
  }

  return { teasers }
}
```

### Why pre-render data

We need to pre-render data on the server whenever we need information to be present immediately inside our initial HTML response. This is useful for making our pages SEO-friendly since we can the content is present in the initial HTML response. In traditional single-page applications, the content of the page usually comes after the initial HTMl is rendered.

### Setting header data and important content on the server

```
<template>
  <div>{{ blog.body }}</div>
</template>
<script>
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
</script>
```

The code above does the following on the server:

1.  Retrieve data using the `asyncData`.
    
2.  Use the data in the `<head>` section of the page.
    
3.  Use the data in the content.
    

Conclusion
----------

Using Firestore seemed like a breeze to me. It is a convenient, low-cost, low-maintenance data storage solution. Its documentation is clear and abundant, and its community vibrant.

Being able to access to the Firebase object from anywhere in the code makes coding a lot easier. I also feel that using `async/await` made the code cleaner and easier to understand.

Finally, if we need content to be pre-rendered, we can use `asyncData`. This is especially useful for making our apps SEO-friendly.