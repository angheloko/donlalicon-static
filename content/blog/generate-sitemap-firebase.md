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
It's almost always a requirement to have a sitemap for your website. Sitemaps help guide search engines crawl your website by telling search engines what pages you think are important.

Objective
---------

I wanted requests to `/sitemap.xml` to return a dynamically generated sitemap that retrieves its data from Firebase.

Firebase Admin
--------------

I used [Firebase Admin](https://firebase.google.com/docs/admin/setup) to retrieve the list of blogs. This is because I wanted to generate the sitemap on the server.

Install the Firebase Admin SDK:

```
npm install firebase-admin
```

Sitemap module
--------------

We will also need the help of the [sitemap module](https://github.com/ekalinin/sitemap.js) to easily generate the sitemap.

Install the sitemap module:

```
npm install sitemap
```

Nuxt serverMiddleware
---------------------

The [serverMiddleware](https://nuxtjs.org/api/configuration-servermiddleware#the-servermiddleware-property) property lets us add custom middleware to handle requests. This allows us to add custom parsers, pre-process data, and even add custom routes.

In order to handle requests to `/sitemap.xml`, I added the custom middleware to the serverMiddleware property of the `nuxt.config.js` like so:

```
serverMiddleware: [
  {
    path: '/sitemap.xml',
    handler: '~/serverMiddleware/sitemap.js'
  }
]
```

Request handler
---------------

For reusability purposes, we created a small Javascript to import and initialise Firebase admin:

`serverMiddleware/firebase-admin.js`

```
const admin = require('firebase-admin')
module.exports = admin.initializeApp({
  credential: admin.credential.applicationDefault()
})
```

I initialised the Firebase Admin SDK using the [default application credentials](https://firebase.google.com/docs/admin/setup#initialize-sdk). This will work if your app is hosted on Google App Engine.

Alternatively, you can generate a service account key and set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable, or you can explicitly pass the path to the service account key in code.

Below is the main code that will generate our sitemap:

`serverMiddleware/sitemap.js`

```
const { createGzip } = require('zlib')
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
    .then((querySnapshot) => {
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

      streamToPromise(pipeline).then((buffer) => {
        res.writeHead(200, {
          'Content-Type': 'application/xml',
          'Content-Encoding': 'gzip'
        })
        res.end(buffer)
      })
    })
}
```

This code does a number of things:

1.  Query the "blogs" collection for all published articles and sort the result by the creation date.
    
2.  Add the records to the sitemap.
    
3.  Return the sitemap with the correct headers.
    

Result
------

The result is a [well-built sitemap](https://donlalicon.dev/sitemap.xml) that automatically gets updated whenever I publish new blogs.