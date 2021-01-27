---
title: "Generate Dynamic Sitemap With Firebase Admin for Your Universal Nuxt App"
subtitle: "How to generate a sitemap with data from Firebase using Firebase Admin"
lead: "Generate a sitemap with data from Firebase using Firebase Admin"
description: "How to generate a sitemap with data from Firebase using Firebase Admin"
createdAt: 2020-01-05T06:49:52.484Z
updatedAt: 2020-01-05T07:06:26.151Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/generate-sitemap-firebase%2Fclint-adair-BW0vK-FA3eg-unsplash.jpg?alt=media&token=08bf8931-90d6-4f2f-aef6-07753add4c80"
  alt: "Generate Dynamic Sitemap With Firebase Admin for Your Universal Nuxt App cover image"
  caption: "Photo by Clint Adair on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/generate-sitemap-firebase%2Fclint-adair-BW0vK-FA3eg-unsplash-small.jpg?alt=media&token=c12c8bdc-7a19-4234-aeee-202d770e4ade"
tags: 
  - Firebase Admin
  - Firestore
  - Nuxt
  - Sitemap
  - SSR
---
<p>It's almost always a requirement to have a sitemap for your website. Sitemaps help guide search engines crawl your website by telling search engines what pages you think are important.</p><h2>Objective</h2><p>I wanted requests to <code>/sitemap.xml</code> to return a dynamically generated sitemap that retrieves its data from Firebase.</p><h2>Firebase Admin</h2><p>I used <a href="https://firebase.google.com/docs/admin/setup" rel="noopener noreferrer nofollow">Firebase Admin</a> to retrieve the list of blogs. This is because I wanted to generate the sitemap on the server.</p><p>Install the Firebase Admin SDK:</p><pre><code>npm install firebase-admin</code></pre><h2>Sitemap module</h2><p>We will also need the help of the <a href="https://github.com/ekalinin/sitemap.js" rel="noopener noreferrer nofollow">sitemap module</a> to easily generate the sitemap.</p><p>Install the sitemap module:</p><pre><code>npm install sitemap</code></pre><h2>Nuxt serverMiddleware</h2><p>The <a href="https://nuxtjs.org/api/configuration-servermiddleware#the-servermiddleware-property" rel="noopener noreferrer nofollow">serverMiddleware</a> property lets us add custom middleware to handle requests. This allows us to add custom parsers, pre-process data, and even add custom routes.</p><p>In order to handle requests to <code>/sitemap.xml</code>, I added the custom middleware to the serverMiddleware property of the <code>nuxt.config.js</code> like so:</p><pre><code>serverMiddleware: [
  {
    path: '/sitemap.xml',
    handler: '~/serverMiddleware/sitemap.js'
  }
]</code></pre><h2>Request handler</h2><p>For reusability purposes, we created a small Javascript to import and initialise Firebase admin:</p><p><code>serverMiddleware/firebase-admin.js</code></p><pre><code>const admin = require('firebase-admin')
module.exports = admin.initializeApp({
  credential: admin.credential.applicationDefault()
})</code></pre><p>I initialised the Firebase Admin SDK using the <a href="https://firebase.google.com/docs/admin/setup#initialize-sdk" rel="noopener noreferrer nofollow">default application credentials</a>. This will work if your app is hosted on Google App Engine.</p><p>Alternatively, you can generate a service account key and set the <code>GOOGLE_APPLICATION_CREDENTIALS</code> environment variable, or you can explicitly pass the path to the service account key in code.</p><p>Below is the main code that will generate our sitemap:</p><p><code>serverMiddleware/sitemap.js</code></p><pre><code>const { createGzip } = require('zlib')
const { SitemapStream, streamToPromise } = require('sitemap')
const app = require('./firebase-admin')

export default function (req, res, next) {
  const db = app.firestore()
  const smStream = new SitemapStream({ hostname: 'https://example.com/' })
  const pipeline = smStream.pipe(createGzip())

  db.collection('blogs')
    .where('published', '==', true)
    .orderBy('created', 'desc')
    .get()
    .then((querySnapshot) =&gt; {
      for (const doc of querySnapshot.docs) {
        const data = doc.data()
        smStream.write({
          url: `/blog/${doc.id}`,
          lastmod: data.changed
            .toDate()
            .toISOString()
            .substr(0, 10)
        })
      }

      smStream.end()

      streamToPromise(pipeline).then((buffer) =&gt; {
        res.writeHead(200, {
          'Content-Type': 'application/xml',
          'Content-Encoding': 'gzip'
        })
        res.end(buffer)
      })
    })
}</code></pre><p>This code does a number of things:</p><ol><li><p>Query the "blogs" collection for all published articles and sort the result by the creation date.</p></li><li><p>Add the records to the sitemap.</p></li><li><p>Return the sitemap with the correct headers.</p></li></ol><h2>Result</h2><p>The result is a <a href="https://donlalicon.dev/sitemap.xml" rel="noopener noreferrer nofollow">well-built sitemap</a> that automatically gets updated whenever I publish new blogs.</p>