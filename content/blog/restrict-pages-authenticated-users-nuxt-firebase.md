---
title: "Restrict Pages to Authenticated Users in Nuxt With Firebase"
subtitle: "A common feature for websites is being able to authenticate users and restrict access to certain pages."
lead: "Securing pages in your Nuxt application"
description: "Restrict Pages to Authenticated Users in Nuxt With Firebase"
createdAt: 2020-01-06T05:40:09.367Z
updatedAt: 2020-01-06T09:27:35.518Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/restrict-pages-authenticated-users-nuxt-firebase%2Fisaw-company-PBJnihY7vaY-unsplash.jpg?alt=media&token=8757452f-158c-450d-bdff-286c06f4dbf6"
  alt: "Restrict Pages to Authenticated Users in Nuxt With Firebase cover image"
  caption: "Photo by iSAW Company on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/restrict-pages-authenticated-users-nuxt-firebase%2Fisaw-company-PBJnihY7vaY-unsplash-small.jpeg?alt=media&token=0ef80da7-5e87-4310-8fa0-89e22d73ed59"
tags: 
  - Firebase Authentication
  - Nuxt
---
A common feature for websites is being able to authenticate users and restrict access to certain pages. My blog, for instance, has a page to manage all the content and this can only be accessed by authenticated users like myself.

[Firebase Authentication](https://firebase.google.com/docs/auth) provides an easy way to store and authenticate your users, and integrating it with your Nuxt application is fairly straightforward.

cookieparser
-------------

We will need the help of the [cookieparser](https://www.npmjs.com/package/cookieparser) module to easily read cookie values on the server. While it seems that the module is meant to be used with [Express](https://expressjs.com/), we can safely use it since Nuxt uses [Connect](https://github.com/senchalabs/connect), which is compatible with the module.

To install the cookie-parser module:

```
npm i cookieparser
```

Firebase plugin
---------------

In a [previous article](https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase), we created a plugin to integrate our Nuxt application with Firebase.

`plugins/firebase.js`:

```js[plugins/firebase.js]
import firebase from 'firebase/app'

// Add the Firebase products that you want to use
import 'firebase/firestore'
import 'firebase/auth'

export default ({ env, store }, inject) => {
  const firebaseConfig = {
    apiKey: env.FB_API_KEY,
    authDomain: env.FB_AUTH_DOMAIN,
    databaseURL: env.FB_DB_URL,
    projectId: env.FB_PROJECT_ID,
    storageBucket: env.FB_STORAGE_BUCKET,
    messagingSenderId: env.FB_MESSAGING_SENDER_ID,
    appId: env.FB_APP_ID,
    measurementId: env.FB_MEASUREMENT_ID
  }

  if (!firebase.apps.length) {
    // Initialize Firebase
    firebase.initializeApp(firebaseConfig)
  }

  if (process.client) {
    firebase.auth().onAuthStateChanged((user) => {
      store.dispatch('setAuth', user)
    })
  }

  inject('firebase', firebase)
}
```

We only need to ensure that the [Authentication module](https://firebase.google.com/docs/web/setup#available-libraries) is being included in our app and that we set an observer to get the current user's authentication state.

According to the [Firebase documentation](https://firebase.google.com/docs/auth/web/manage-users#get_the_currently_signed-in_user), the recommended way to get the current user is by setting an observer. Each time a change is made on the user object, our callback will be executed.

Every time the user object changes, we'll dispatch a [Vuex action](https://vuex.vuejs.org/guide/actions.html), `setAuth`, which basically stores the current user's token.

Notice that we're also only setting the observer when running on the client. This is because Firebase Authentication SDK requires a full-pledge browser and as such can only run on the client.

Vuex store
----------

The best way to store application state is via [Vuex](https://vuex.vuejs.org/guide/). Nuxt integrates Vuex at its core so there is no need to manually install it.

`store/index.js`:

```js[store/index.js]
const cookieParser = process.server ? require('cookieparser') : undefined

export const state = () => {
  return {
    auth: null
  }
}
export const getters = {
  auth: (state) => {
    return state.auth
  }
}
export const mutations = {
  setAuth (state, auth) {
    state.auth = auth
  }
}
export const actions = {
  nuxtServerInit ({ commit }, { req }) {
    // Get the token from the cookie, if available, and
    // initialise the `auth` state with it.
    let token = null
    if (req.headers.cookie) {
      const parsed = cookieParser.parse(req.headers.cookie)
      token = parsed.token
    }
    commit('setAuth', token)
  },
  setAuth ({ commit }, user) {
    if (!user) {
      commit('setAuth', null)
      document.cookie = 'token=;path=/;expires=0'
    } else {
      // When we have a valid user, we retrieve the user's token and save it to the `auth` state as well as in a cookie.
      user.getIdToken().then((token) => {
        commit('setAuth', token)
        const expiresIn = 60 * 60 * 24 * 5 * 1000 // 5 days.
        document.cookie = 'token=' + token + ';path=/;max-age=' + expiresIn
      }, (error) => {
        console.log('Error getting ID token.', error)
        commit('setAuth', null)
        document.cookie = 'token=;path=/;expires=0'
      })
    }
  }
}
```

### setAuth action

This action gets called whenever there is a change in the user object.

If we have a valid user, we retrieve the user's token (JSON Web Token) using the `getIdToken` method. We also need to create a cookie to store the token so that the token can be passed to the server. This allows us to determine the user's authentication state even on the server.

### nuxtServerInit action

This action allows us to initialise our store from the server. We extract the token from the cookie and initialise our `auth` state with it. This allows the authentication state to persist even when the user refreshes the page or accesses the application for the first time (i.e. by an external link or by entering the URL directly in the browser).

Our application will still work without this but the authentication will be delayed since we will need to wait for our observer to be triggered on the client, which may be undesirable since restricted pages might be displayed briefly.

Login page
----------

We authenticate users using their [email address and password](https://firebase.google.com/docs/auth/web/password-auth). The login page contains a simple form where the user can enter their email address and password. When the user clicks on the "Sign In" button, we authenticate the user using the `signInWithEmailAndPassword` method.

`pages/login.vue`:

```vue[pages/login.vue]
<template>
  <form @submit.prevent>
    <div>
      <label for="email">Email</label>
      <input id="email" v-model="email" type="text" placeholder="Email">
    </div>
    <div>
      <label for="password">Password</label>
      <input id="password" v-model="password" type="password" placeholder="Password">
    </div>
    <div>
      <button @click="submitForm" type="submit">Sign In</button>
    </div>
  </form>
</template>
<script>
import { mapGetters } from 'vuex'
export default {
  data () {
    return {
      email: '',
      password: ''
    }
  },
  computed: {
    // Map `auth` state to a local property so we can access it locally.
    ...mapGetters(['auth'])
  },
  watch: {
    auth (value) {
      // Redirect the user to the home page once user is authenticated.
      // We know user is authenticated if `auth` contains a value, which
      // is automatically updated when user's state changes.
      // @see actions.setAuth in store/index.js
      // @see plugins/firebase.js
      if (value) {
        this.$router.push({
          path: '/restricted-page'
        })
      }
    }
  },
  methods: {
    submitForm () {
      // Trigger authentication when user submits the form.
      this.$firebase.auth().signInWithEmailAndPassword(this.email, this.password)
        .catch(function (error) {
          alert(error.message)
        })
    }
  }
}
</script>
```

Aside from authenticating the user, we are also using Vuex [**mapGetters**](https://vuex.vuejs.org/guide/getters.html#the-mapgetters-helper) to access the `auth` state locally as a computed property. This lets us [watch](https://vuejs.org/v2/guide/computed.html#Watchers) the `auth` state. If the `auth` state is no longer empty, we can redirect the user back to the restricted page.

Middleware
----------

`middleware/authenticated-access.js`:

```js[middleware/authenticated-access.js]
export default function ({ store, redirect }) {
  if (!store.state.auth) {
    return redirect('/login')
  }
}
```

So this is pretty straightforward. We check the store if the `auth` state is set. If it's not, we redirect the user to our login page.

Restricted pages
----------------

Now that we have a form for users to log in, a way to store the current user's state, and a middleware to check the authentication state, we can now secure our restricted pages.

To restrict access to a page, we simply need to set the [middleware property](https://nuxtjs.org/api/pages-middleware#the-middleware-property) of the page to the name of the middleware.

Example `pages/restricted.vue`:

```vue[pages/restricted.vue]
<template>
  <div>
    This is a restricted page.
  </div>
</template>
<script>
export default {
  middleware: 'authenticated-access'
}
</script>
```

Final words
-----------

There are other ways to authenticate the user aside from using their email address and password. You can use [Google](https://firebase.google.com/docs/auth/web/google-signin), [Facebook](https://firebase.google.com/docs/auth/web/facebook-login) and [Twitter](https://firebase.google.com/docs/auth/web/twitter-login), and more. Regardless of the authentication provider you choose, the process will be similar.