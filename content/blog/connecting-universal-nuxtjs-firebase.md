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
<p>Getting your application to start communicating with Firebase is pretty easy.</p><h2>Nuxt Plugins</h2><p><a href="https://nuxtjs.org/guide/plugins" rel="noopener noreferrer nofollow">Plugins</a> let you set up external modules or packages, or run custom Javascript code before the root Vue.js application is instantiated. Think of it as a way of introducing your application to other modules or packages, or custom code, the <em>Nuxt way</em>.</p><p>Essentially, we want to be able to use the <a href="https://firebase.google.com/docs/web/setup" rel="noopener noreferrer nofollow">Firebase Javascript library</a> and to do so we must first initialise it and make it <em>known</em> to our Nuxt application and the best way to do this is through plugins.</p><p>The plugin file <code>plugins/firebase.js</code>:</p><pre><code>import firebase from 'firebase/app'
import 'firebase/firestore'

export default ({ env, store }, inject) =&gt; {
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
}</code></pre><p>There are a number of things that we're doing here:</p><ol><li><p>We <a href="https://firebase.google.com/docs/web/setup#config-object" rel="noopener noreferrer nofollow">initialise the Firebase SDK</a> by providing our app's Firebase project configuration.</p></li><li><p>We only include the <a href="https://firebase.google.com/docs/web/setup#namespace" rel="noopener noreferrer nofollow">Firebase features</a> that we need, which is <code>firestore</code>.</p></li><li><p>And since we want to have access to the Firebase object across the app (e.g. <code>context</code>, Vue components, Vuex store), we <a href="https://nuxtjs.org/guide/plugins#combined-inject" rel="noopener noreferrer nofollow">inject the Firebase object</a> into the context as well as into the Vue instances using the <code>inject</code> function.</p></li></ol><h2>Nuxt Configuration</h2><p>After creating the file, we need to make Nuxt aware of it by adding it into our <code>nuxt.config.js</code> file:</p><pre><code>/*
** Plugins to load before mounting the App
*/
plugins: [
  '~/plugins/firebase'
],</code></pre><h2>Usage</h2><p>Since we made the Firebase object accessible across the app, using it becomes straightforward. Depending on where we need a Firebase feature, we might access it using the <code>context</code> or <code>this</code> objects</p><h3>With context</h3><p>Whenever you have access to the <code>context</code> object like in the <code>asyncData</code> method, you can access the Firebase object like so:</p><pre><code>export default {
  async asyncData (context) {
    const db = context.app.$firebase.firestore()
  }
}</code></pre><p>Or when <code>context</code> parameter is deconstructed:</p><pre><code>export default {
  async asyncData ({ app }) {
    const db = app.$firebase.firestore()
  }
}</code></pre><h3>In Vue instances</h3><p>Once within the Vue instance lifecycle you can access the Firebase object via <code>this</code>:</p><pre><code>export default {
  mounted () {
    const db = this.$firebase.firestore()
  }
}</code></pre><h3>In Vuex store</h3><p>You can also access the Firebase object via <code>this</code> when in the Vuex store:</p><pre><code>export const mutations = {
  changeSomeValue (state, newValue) {
    const db = this.$firebase.firestore()
  }
}</code></pre><h2>Conclusion</h2><p>That's all there is to it!</p><p>Of course, there are other ways of integrating Firebase with a Nuxt application such as using community contributed open-source modules or creating a custom standalone Firebase service library that you can just include as and when needed.</p><p>Personally, I find that this is the most straightforward and closest to the <em>Nuxt</em> <em>way </em>of doing things. In the next articles, we'll be looking into the various features of Firebase, like <a href="https://firebase.google.com/docs/firestore" rel="noopener noreferrer nofollow">Firestore</a>, <a href="https://firebase.google.com/docs/auth" rel="noopener noreferrer nofollow">Authentication</a>, and <a href="https://firebase.google.com/docs/storage" rel="noopener noreferrer nofollow">Storage</a>.</p>