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
<p>You can add custom attributes, or <a href="https://firebase.google.com/docs/auth/admin/custom-claims" rel="noopener noreferrer nofollow">custom claims</a>, to your users in your Firebase apps, which then lets you implement your own access control strategies, like role-based or attribute-based access control, depending on your needs.</p><h2>Custom claims</h2><p>Custom claims are essentially additional data about your user that is stored in the user's ID token. The ID token is a JSON Web Token (JWT) used to identify the user.</p><h2>Setting custom claims</h2><p>There are <a href="https://firebase.google.com/docs/auth/admin/custom-claims#examples_and_use_cases" rel="noopener noreferrer nofollow">various ways</a> on how you can set custom claims but my favorite is by setting the custom claims via an HTTP request since this lets me set the custom claims dynamically.</p><p>This approach is perfect for applications built with Nuxt.js mainly because of its built-in support for <a href="https://nuxtjs.org/api/configuration-servermiddleware/" rel="noopener noreferrer nofollow">server-side middlewares</a>. Because of this feature, we can create custom web APIs, among other things such as <a href="https://donlalicon.dev/blog/restrict-pages-authenticated-users-nuxt-firebase" rel="noopener noreferrer nofollow">setting a cookie on the server</a>, without the need for another separate application or an external server.</p><h2>The server middleware</h2><p>You'll need to install <a href="https://expressjs.com/" rel="noopener noreferrer nofollow">Express</a> to make setting up our endpoints easier.</p><p><code>nuxt.config.js</code></p><p>Define the custom server middleware in the Nuxt config file. This code lets any HTTP request sent to <code>/set-custom-claims</code> be handled by our custom middleware.</p><pre><code>serverMiddleware: [
  {
    path: '/set-custom-claims',
    handler: '~/serverMiddleware/set-custom-claims'
  }
]</code></pre><p><code>serverMiddleware/firebase-admin.js</code></p><pre><code>const admin = require('firebase-admin')
module.exports = admin.initializeApp({
  credential: admin.credential.applicationDefault()
})</code></pre><p>This is just a way to organize the code. We separate the code responsible for initializing the Firebase Admin into its own module.</p><p><code>serverMiddleware/set-custom-claims.js</code></p><p>This is where we do our heavy lifting. Refer to the comments in the code for details on what it does.</p><pre><code>const express = require('express');
const admin = require('./firebase-admin');

const app = express();

// Let's us read JSON data easily.
app.use(express.json());

app.post('/', (req, res) =&gt; {
  // Get the ID token from the request body.
  const idToken = req.body.idToken;

  // Verify the ID token and decode its payload.
  admin.auth().verifyIdToken(idToken).then((claims) =&gt; {
    // Verify user is eligible for additional privileges.
    if (claims.email &amp;&amp; claims.email.endsWith('@donlalicon.dev')) {

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

module.exports = app;</code></pre><p>That's it for everything server-side!</p><h2>Accessing the claims on the client</h2><p>To access the decoded token on the client, simply call the <a href="https://firebase.google.com/docs/reference/js/firebase.User.html#getidtokenresult" rel="noopener noreferrer nofollow">getIdTokenResult()</a> function:</p><pre><code>firebase.auth().currentUser.getIdTokenResult().then((idTokenResult) =&gt; {
  // Check if the user is an admin.
  if (!!idTokenResult.claims.admin) {
    console.log('User is an admin')
  } else {
    console.log('User is not an admin')
  }
}).catch((error) =&gt; {
  console.log(error)
})</code></pre><h2>Observing token changes and persisting claims</h2><p>It's a good idea to observe token changes so that our application can react to it. For example, when a user's token somehow expires, we need to make sure that the user can no longer access a restricted page.</p><p><code>nuxt.config.js</code></p><p>Define a new plugin. Plugins are code that you want to always run so this is a good place for our observer.</p><pre><code>plugins: [
  {
    src: '~/plugins/firebase-token-change-listener.js',
    mode: 'client'
  }
]</code></pre><p><code>plugins/firebase-token-change-listener.js</code></p><p>Whenever the current user's ID token changes, we dispatch a <a href="https://vuex.vuejs.org/guide/actions.html" rel="noopener noreferrer nofollow">Vuex action</a>.</p><pre><code>import { auth } from '../services/firebase'

export default ({ store }) =&gt; {
  auth.onIdTokenChanged((user) =&gt; {
    store.dispatch('refreshToken', user)
  })
}</code></pre><p><code>services/firebase.js</code></p><p>It's a good idea to organize code in a way that will let us reuse them.</p><pre><code>import firebase from 'firebase/app'
import 'firebase/auth'

if (!firebase.apps.length) {
  const config = process.env.firebaseConfig
  firebase.initializeApp(config)
}

export const auth = firebase.auth()
export default firebase</code></pre><p><code>store/index.js</code></p><p>This is where we handle storage of our user's custom claims. The <code>refreshToken</code> action is called whenever the current user's ID token changes.</p><pre><code>export const state = () =&gt; ({
  admin: false
})

export const getters = {
  isAdmin: (state) =&gt; {
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
      user.getIdTokenResult().then((idTokenResult) =&gt; {
        // Get the decoded claims.
        const { claims } = idTokenResult
        // Get the "admin" attribute.
        const { admin } = claims
        commit('setAdmin', admin)
      })
    }
  }
}</code></pre><h2>Accessing the store</h2><p>Now that we store the current user's <code>admin</code> attribute via <a href="https://vuex.vuejs.org/" rel="noopener noreferrer nofollow">Vuex</a>, checking if the current user is an admin or not becomes very easy especially with Vuex's helper functions like <a href="https://vuex.vuejs.org/guide/getters.html#the-mapgetters-helper" rel="noopener noreferrer nofollow">mapGetters()</a>.</p><h3>In a component or page</h3><pre><code>import { mapGetters } from 'vuex'

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
}</code></pre><h3>As a middleware for guarding restricted pages</h3><p>This is useful if you want to guard restricted pages. Page <a href="https://nuxtjs.org/api/pages-middleware/" rel="noopener noreferrer nofollow">middlewares</a> are called whenever you visit the page.</p><p><code>middleware/admin-guard.js</code></p><pre><code>export default ({ store, error }) =&gt; {
  if (!!store.state.admin) {
    error({
      message: 'Restricted area',
      statusCode: 403
    })
  }
}</code></pre><p><code>pages/admin.vue</code></p><pre><code>&lt;template&gt;
  &lt;div&gt;
    Admin only area
  &lt;/div&gt;
&lt;/template&gt;

&lt;script&gt;
export default {
  middleware: 'admin-guard'
}
&lt;/script&gt;</code></pre><p></p>