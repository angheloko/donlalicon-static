---
title: "Connecting Your Universal Nuxt Application with Firebase"
subtitle: "How to connect your universal Nuxt application with Firebase"
lead: "Supercharge your Nuxt applications with Firebase"
description: "How to integrate your Nuxt SSR Universal app with Firebase"
createdAt: 2020-01-02T05:14:31.641Z
updatedAt: 2020-01-05T01:44:44.067Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/connecting-universal-nuxtjs-firebase%2Ftoa-heftiba-rdoRdjOk-OY-unsplash.jpg?alt=media&token=944030b1-662e-46b3-b5b6-a9b05292abde"
  alt: "Connecting Your Universal Nuxt Application with Firebase cover image"
  caption: "Photo by Toa Heftiba on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/connecting-universal-nuxtjs-firebase%2Ftoa-heftiba-rdoRdjOk-OY-unsplash-small.jpg?alt=media&token=854f6c22-32af-489f-a007-81f073330287"
tags: 
  - Firebase
  - Nuxt
  - SSR
---
Getting your application to start communicating with Firebase is pretty easy.

Nuxt Plugins
------------

[Plugins](https://nuxtjs.org/guide/plugins) let you set up external modules or packages, or run custom Javascript code before the root Vue.js application is instantiated. Think of it as a way of introducing your application to other modules or packages, or custom code, the _Nuxt way_.

Essentially, we want to be able to use the [Firebase Javascript library](https://firebase.google.com/docs/web/setup) and to do so we must first initialise it and make it _known_ to our Nuxt application and the best way to do this is through plugins.

The plugin file `plugins/firebase.js`:

```
import firebase from 'firebase/app'
import 'firebase/firestore'

export default ({ env, store }, inject) => {
  const firebaseConfig = {
    apiKey: 'api-key',
    authDomain: 'project-id.firebaseapp.com',
    databaseURL: 'https://project-id.firebaseio.com',
    projectId: 'project-id',
    storageBucket: 'project-id.appspot.com',
    messagingSenderId: 'sender-id',
    appId: 'app-id',
    measurementId: 'G-measurement-id'
  }

  if (!firebase.apps.length) {
    // Initialize Firebase
    firebase.initializeApp(firebaseConfig)
  }

  inject('firebase', firebase)
}
```

There are a number of things that we're doing here:

1.  We [initialise the Firebase SDK](https://firebase.google.com/docs/web/setup#config-object) by providing our app's Firebase project configuration.
    
2.  We only include the [Firebase features](https://firebase.google.com/docs/web/setup#namespace) that we need, which is `firestore`.
    
3.  And since we want to have access to the Firebase object across the app (e.g. `context`, Vue components, Vuex store), we [inject the Firebase object](https://nuxtjs.org/guide/plugins#combined-inject) into the context as well as into the Vue instances using the `inject` function.
    

Nuxt Configuration
------------------

After creating the file, we need to make Nuxt aware of it by adding it into our `nuxt.config.js` file:

```
/*
** Plugins to load before mounting the App
*/
plugins: [
  '~/plugins/firebase'
],
```

Usage
-----

Since we made the Firebase object accessible across the app, using it becomes straightforward. Depending on where we need a Firebase feature, we might access it using the `context` or `this` objects

### With context

Whenever you have access to the `context` object like in the `asyncData` method, you can access the Firebase object like so:

```
export default {
  async asyncData (context) {
    const db = context.app.$firebase.firestore()
  }
}
```

Or when `context` parameter is deconstructed:

```
export default {
  async asyncData ({ app }) {
    const db = app.$firebase.firestore()
  }
}
```

### In Vue instances

Once within the Vue instance lifecycle you can access the Firebase object via `this`:

```
export default {
  mounted () {
    const db = this.$firebase.firestore()
  }
}
```

### In Vuex store

You can also access the Firebase object via `this` when in the Vuex store:

```
export const mutations = {
  changeSomeValue (state, newValue) {
    const db = this.$firebase.firestore()
  }
}
```

Conclusion
----------

That's all there is to it!

Of course, there are other ways of integrating Firebase with a Nuxt application such as using community contributed open-source modules or creating a custom standalone Firebase service library that you can just include as and when needed.

Personally, I find that this is the most straightforward and closest to the _Nuxt_ _way_ of doing things. In the next articles, we'll be looking into the various features of Firebase, like [Firestore](https://firebase.google.com/docs/firestore), [Authentication](https://firebase.google.com/docs/auth), and [Storage](https://firebase.google.com/docs/storage).