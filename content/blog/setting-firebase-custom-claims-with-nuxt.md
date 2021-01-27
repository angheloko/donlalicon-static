---
title: "Setting Firebase Custom Claims With Nuxt"
subtitle: "How to set and use Firebase Custom Claims in your Nuxt application"
lead: "How to set and use Firebase Custom Claims in your Nuxt application"
description: ""
createdAt: 2020-01-23T08:25:26.711Z
updatedAt: 2020-01-23T08:53:08.947Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/setting-firebase-custom-claims-with-nuxt%2Fcustomers-users-color-wheel-6231.jpg?alt=media&token=6ec00541-b908-42aa-a1c2-9d0975ca88f8"
  alt: "Setting Firebase Custom Claims With Nuxt cover image"
  caption: "Photo by Kaboompics .com from Pexels"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/setting-firebase-custom-claims-with-nuxt%2Fcustomers-users-color-wheel-6231_200x200.jpg?alt=media"
tags: 
  - Firebase Custom Claims
  - Nuxt
  - Vuex
---
You can add custom attributes, or [custom claims](https://firebase.google.com/docs/auth/admin/custom-claims), to your users in your Firebase apps, which then lets you implement your own access control strategies, like role-based or attribute-based access control, depending on your needs.

Custom claims
-------------

Custom claims are essentially additional data about your user that is stored in the user's ID token. The ID token is a JSON Web Token (JWT) used to identify the user.

Setting custom claims
---------------------

There are [various ways](https://firebase.google.com/docs/auth/admin/custom-claims#examples_and_use_cases) on how you can set custom claims but my favorite is by setting the custom claims via an HTTP request since this lets me set the custom claims dynamically.

This approach is perfect for applications built with Nuxt.js mainly because of its built-in support for [server-side middlewares](https://nuxtjs.org/api/configuration-servermiddleware/). Because of this feature, we can create custom web APIs, among other things such as [setting a cookie on the server](https://donlalicon.dev/blog/restrict-pages-authenticated-users-nuxt-firebase), without the need for another separate application or an external server.

The server middleware
---------------------

You'll need to install [Express](https://expressjs.com/) to make setting up our endpoints easier.

`nuxt.config.js`

Define the custom server middleware in the Nuxt config file. This code lets any HTTP request sent to `/set-custom-claims` be handled by our custom middleware.

```
serverMiddleware: [
  {
    path: '/set-custom-claims',
    handler: '~/serverMiddleware/set-custom-claims'
  }
]
```

`serverMiddleware/firebase-admin.js`

```
const admin = require('firebase-admin')
module.exports = admin.initializeApp({
  credential: admin.credential.applicationDefault()
})
```

This is just a way to organize the code. We separate the code responsible for initializing the Firebase Admin into its own module.

`serverMiddleware/set-custom-claims.js`

This is where we do our heavy lifting. Refer to the comments in the code for details on what it does.

```
const express = require('express');
const admin = require('./firebase-admin');

const app = express();

// Let's us read JSON data easily.
app.use(express.json());

app.post('/', (req, res) => {
  // Get the ID token from the request body.
  const idToken = req.body.idToken;

  // Verify the ID token and decode its payload.
  admin.auth().verifyIdToken(idToken).then((claims) => {
    // Verify user is eligible for additional privileges.
    if (claims.email && claims.email.endsWith('@donlalicon.dev')) {

      // Add any arbitrary data to the custom claims.
      // In this case, we're adding an "admin" attribute.
      const customClaims = {
        admin: true
      }

      // Set the custom claims.
      admin
        .auth()
        .setCustomUserClaims(claims.uid, customClaims)
        .then(function() {
          // Tell client to refresh token on user.
          res.end(JSON.stringify({status: 'success'}));
        });
    } else {
      // Tell client there are no custom claims added.
      res.end(JSON.stringify({status: 'ineligible'}));
    }
  });
});

module.exports = app;
```

That's it for everything server-side!

Accessing the claims on the client
----------------------------------

To access the decoded token on the client, simply call the [getIdTokenResult()](https://firebase.google.com/docs/reference/js/firebase.User.html#getidtokenresult) function:

```
firebase.auth().currentUser.getIdTokenResult().then((idTokenResult) => {
  // Check if the user is an admin.
  if (!!idTokenResult.claims.admin) {
    console.log('User is an admin')
  } else {
    console.log('User is not an admin')
  }
}).catch((error) => {
  console.log(error)
})
```

Observing token changes and persisting claims
---------------------------------------------

It's a good idea to observe token changes so that our application can react to it. For example, when a user's token somehow expires, we need to make sure that the user can no longer access a restricted page.

`nuxt.config.js`

Define a new plugin. Plugins are code that you want to always run so this is a good place for our observer.

```
plugins: [
  {
    src: '~/plugins/firebase-token-change-listener.js',
    mode: 'client'
  }
]
```

`plugins/firebase-token-change-listener.js`

Whenever the current user's ID token changes, we dispatch a [Vuex action](https://vuex.vuejs.org/guide/actions.html).

```
import { auth } from '../services/firebase'

export default ({ store }) => {
  auth.onIdTokenChanged((user) => {
    store.dispatch('refreshToken', user)
  })
}
```

`services/firebase.js`

It's a good idea to organize code in a way that will let us reuse them.

```
import firebase from 'firebase/app'
import 'firebase/auth'

if (!firebase.apps.length) {
  const config = process.env.firebaseConfig
  firebase.initializeApp(config)
}

export const auth = firebase.auth()
export default firebase
```

`store/index.js`

This is where we handle storage of our user's custom claims. The `refreshToken` action is called whenever the current user's ID token changes.

```
export const state = () => ({
  admin: false
})

export const getters = {
  isAdmin: (state) => {
    return state.admin
  }
}

export const mutations = {
  setAdmin(state, isAdmin) {
    state.admin = !!isAdmin
  }
}

export const actions = {
  refreshToken({ commit }, user) {
    if (!user) {
      commit('setAdmin', false)
    } else {
      user.getIdTokenResult().then((idTokenResult) => {
        // Get the decoded claims.
        const { claims } = idTokenResult
        // Get the "admin" attribute.
        const { admin } = claims
        commit('setAdmin', admin)
      })
    }
  }
}
```

Accessing the store
-------------------

Now that we store the current user's `admin` attribute via [Vuex](https://vuex.vuejs.org/), checking if the current user is an admin or not becomes very easy especially with Vuex's helper functions like [mapGetters()](https://vuex.vuejs.org/guide/getters.html#the-mapgetters-helper).

### In a component or page

```
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters(['isAdmin'])
  },
  watch: {
    isAdmin(value) {
      // This will be called whenever the admin state changes.
    }
  },
  mounted() {
    // You can check if the current user is an admin by accessing this.isAdmin.
  }
}
```

### As a middleware for guarding restricted pages

This is useful if you want to guard restricted pages. Page [middlewares](https://nuxtjs.org/api/pages-middleware/) are called whenever you visit the page.

`middleware/admin-guard.js`

```
export default ({ store, error }) => {
  if (!!store.state.admin) {
    error({
      message: 'Restricted area',
      statusCode: 403
    })
  }
}
```

`pages/admin.vue`

```
<template>
  <div>
    Admin only area
  </div>
</template>

<script>
export default {
  middleware: 'admin-guard'
}
</script>
```