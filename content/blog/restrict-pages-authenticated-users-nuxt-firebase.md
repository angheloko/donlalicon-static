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
<p>A common feature for websites is being able to authenticate users and restrict access to certain pages. My blog, for instance, has a page to manage all the content and this can only be accessed by authenticated users like myself.</p><p><a href="https://firebase.google.com/docs/auth" rel="noopener noreferrer nofollow">Firebase Authentication</a> provides an easy way to store and authenticate your users, and integrating it with your Nuxt application is fairly straightforward.</p><h2>cookie-parser</h2><p>We will need the help of the <a href="https://github.com/expressjs/cookie-parser" rel="noopener noreferrer nofollow">cookie-parser</a> module to easily read cookie values on the server. While it seems that the module is meant to be used with <a href="https://expressjs.com/" rel="noopener noreferrer nofollow">Express</a>, we can safely use it since Nuxt uses <a href="https://github.com/senchalabs/connect" rel="noopener noreferrer nofollow">Connect</a>, which is compatible with the module.</p><p>To install the cookie-parser module:</p><pre><code>npm i cookie-parser</code></pre><h2>Firebase plugin</h2><p>In a <a href="https://donlalicon.dev/blog/connecting-universal-nuxtjs-firebase" rel="noopener noreferrer nofollow">previous article</a>, we created a plugin to integrate our Nuxt application with Firebase.</p><p><code>plugins/firebase.js</code>:</p><pre><code>import firebase from 'firebase/app'

// Add the Firebase products that you want to use
import 'firebase/firestore'
import 'firebase/auth'

export default ({ env, store }, inject) =&gt; {
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
    firebase.auth().onAuthStateChanged((user) =&gt; {
      store.dispatch('setAuth', user)
    })
  }

  inject('firebase', firebase)
}
</code></pre><p>We only need to ensure that the <a href="https://firebase.google.com/docs/web/setup#available-libraries" rel="noopener noreferrer nofollow">Authentication module</a> is being included in our app and that we set an observer to get the current user's authentication state.</p><p>According to the <a href="https://firebase.google.com/docs/auth/web/manage-users#get_the_currently_signed-in_user" rel="noopener noreferrer nofollow">Firebase documentation</a>, the recommended way to get the current user is by setting an observer. Each time a change is made on the user object, our callback will be executed.</p><p>Every time the user object changes, we'll dispatch a <a href="https://vuex.vuejs.org/guide/actions.html" rel="noopener noreferrer nofollow">Vuex action</a>, <code>setAuth</code>, which basically stores the current user's token.</p><p>Notice that we're also only setting the observer when running on the client. This is because Firebase Authentication SDK requires a full-pledge browser and as such can only run on the client.</p><h2>Vuex store</h2><p>The best way to store application state is via <a href="https://vuex.vuejs.org/guide/" rel="noopener noreferrer nofollow">Vuex</a>. Nuxt integrates Vuex at its core so there is no need to manually install it.</p><p><code>store/index.js</code>:</p><pre><code>const cookieParser = process.server ? require('cookieparser') : undefined

export const state = () =&gt; {
  return {
    auth: null
  }
}
export const getters = {
  auth: (state) =&gt; {
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
      user.getIdToken().then((token) =&gt; {
        commit('setAuth', token)
        const expiresIn = 60 * 60 * 24 * 5 * 1000 // 5 days.
        document.cookie = 'token=' + token + ';path=/;max-age=' + expiresIn
      }, (error) =&gt; {
        console.log('Error getting ID token.', error)
        commit('setAuth', null)
        document.cookie = 'token=;path=/;expires=0'
      })
    }
  }
}</code></pre><h3>setAuth action</h3><p>This action gets called whenever there is a change in the user object.</p><p>If we have a valid user, we retrieve the user's token (JSON Web Token) using the <code>getIdToken</code> method. We also need to create a cookie to store the token so that the token can be passed to the server. This allows us to determine the user's authentication state even on the server.</p><h3>nuxtServerInit action</h3><p>This action allows us to initialise our store from the server. We extract the token from the cookie and initialise our <code>auth</code> state with it. This allows the authentication state to persist even when the user refreshes the page or accesses the application for the first time (i.e. by an external link or by entering the URL directly in the browser).</p><p>Our application will still work without this but the authentication will be delayed since we will need to wait for our observer to be triggered on the client, which may be undesirable since restricted pages might be displayed briefly.</p><h2>Login page</h2><p>We authenticate users using their <a href="https://firebase.google.com/docs/auth/web/password-auth" rel="noopener noreferrer nofollow">email address and password</a>. The login page contains a simple form where the user can enter their email address and password. When the user clicks on the "Sign In" button, we authenticate the user using the <code>signInWithEmailAndPassword</code> method.</p><p><code>pages/login.vue</code>:</p><pre><code>&lt;template&gt;
  &lt;form @submit.prevent&gt;
    &lt;div&gt;
      &lt;label for="email"&gt;Email&lt;/label&gt;
      &lt;input id="email" v-model="email" type="text" placeholder="Email"&gt;
    &lt;/div&gt;
    &lt;div&gt;
      &lt;label for="password"&gt;Password&lt;/label&gt;
      &lt;input id="password" v-model="password" type="password" placeholder="Password"&gt;
    &lt;/div&gt;
    &lt;div&gt;
      &lt;button @click="submitForm" type="submit"&gt;Sign In&lt;/button&gt;
    &lt;/div&gt;
  &lt;/form&gt;
&lt;/template&gt;
&lt;script&gt;
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
&lt;/script&gt;
</code></pre><p>Aside from authenticating the user, we are also using Vuex <a href="https://vuex.vuejs.org/guide/getters.html#the-mapgetters-helper" rel="noopener noreferrer nofollow"><strong>mapGetters</strong></a> to access the <code>auth</code> state locally as a computed property. This lets us <a href="https://vuejs.org/v2/guide/computed.html#Watchers" rel="noopener noreferrer nofollow">watch</a> the <code>auth</code> state. If the <code>auth</code> state is no longer empty, we can redirect the user back to the restricted page.</p><h2>Middleware</h2><p><code>middleware/authenticated-access.js</code>:</p><pre><code>export default function ({ store, redirect }) {
  if (!store.state.auth) {
    return redirect('/login')
  }
}</code></pre><p>So this is pretty straightforward. We check the store if the <code>auth</code> state is set. If it's not, we redirect the user to our login page.</p><h2>Restricted pages</h2><p>Now that we have a form for users to log in, a way to store the current user's state, and a middleware to check the authentication state, we can now secure our restricted pages.</p><p>To restrict access to a page, we simply need to set the <a href="https://nuxtjs.org/api/pages-middleware#the-middleware-property" rel="noopener noreferrer nofollow">middleware property</a> of the page to the name of the middleware.</p><p>Example <code>pages/restricted.vue</code>:</p><pre><code>&lt;template&gt;
  &lt;div&gt;
    This is a restricted page.
  &lt;/div&gt;
&lt;/template&gt;
&lt;script&gt;
export default {
  middleware: 'authenticated-access'
}
&lt;/script&gt;</code></pre><h2>Final words</h2><p>There are other ways to authenticate the user aside from using their email address and password. You can use <a href="https://firebase.google.com/docs/auth/web/google-signin" rel="noopener noreferrer nofollow">Google</a>, <a href="https://firebase.google.com/docs/auth/web/facebook-login" rel="noopener noreferrer nofollow">Facebook</a> and <a href="https://firebase.google.com/docs/auth/web/twitter-login" rel="noopener noreferrer nofollow">Twitter</a>, and more. Regardless of the authentication provider you choose, the process will be similar.</p>