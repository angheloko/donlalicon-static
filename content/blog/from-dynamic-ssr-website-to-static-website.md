---
title: From Dynamic SSR Website to Static Website 
subtitle: Where I refactored my blog from an SSR application using Cloud Firestore to a static application
lead: It's a new year, and I felt like it was time to try something new.
cover:
  image: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/ian-schneider-PAykYb-8Er8-unsplash-1280.jpg?alt=media&token=f980b83f-74f2-4e64-bf36-af81824d8a61
  alt: New year, new site
  caption: '<span>Photo by <a href="https://unsplash.com/@goian?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ian Schneider</a> on <a href="https://unsplash.com/s/photos/new?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'
  thumb: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/ian-schneider-PAykYb-8Er8-unsplash-640.jpg?alt=media&token=ff74111f-4a68-4dd4-a3d3-292c97ad7aa1
---
## Introduction
My blog's content was originally managed via [Cloud Firestore](https://firebase.google.com/docs/firestore). There is definitely nothing wrong with this approach, and for medium to large applications, I would absolutely recommend using it. However, for small personal sites such as mine, a *static content* driven approach might be more suitable.

## Challenges
There are basically 2 main problems involved in converting a dynamic content driven site into a static one:

1. Migrating the data
2. Generating the static website

## Generating static websites

Probably the easiest of the 2 since there are a number of options out there for generating static websites such as [Gatsby](https://www.gatsbyjs.com/) and [Next.js](https://nextjs.org/). Due to my familiarity with it, and since my website was originally built with it, I went with the easy choice of using [Nuxt.js](https://nuxtjs.org/).

### Static content for static websites

Static websites are basically websites that doesn't need to dynamically generate its pages whenever a user visits the site. All the files necessary for all the website's pages to be fully accessible to the user are all pre-built.

For example, my [blog used to retrieve its contents from a database](/blog/get-data-cloud-firestore-universal-nuxt) in order to generate the page before showing it to the user. For static websites, all pages are already built with the content in it.

### Nuxt Content

You *could* create all your blog pages in HTML and you would already have a static website, but writing articles in HTML isn't always a pleasant experience. This is where **Nuxt Content** comes in. 

[Nuxt Content](https://content.nuxtjs.org/) allows your site's content to be stored and retrieved from separate files located in the aptly named `content` directory and will automatically create the HTML pages for you.

## From Cloud Firestore to static content files

My blog's data used to be stored in Cloud Firestore. Going static meant that I had to get those data and store them locally into files so that **Nuxt Content** can retrieve them.  

**This process could be ridiculously straightforward or incredibly hard depending on how you structured your data**. Since my site is just a plain blog site, the data structure was very simple, and thus migrating the data wasn't much of a problem for me.

### The gist of the gist

In the spirit of automation, I [wrote a script](https://gist.github.com/angheloko/7bbfff17c2b74e7b61d3cb048c4ccf6b) that downloads all my blog data into content files ready for Nuxt Content to use.

The script basically does the following:

1. Retrieve all blog data from Cloud Firestore using the [Firebase Admin SDK for Node.js](https://firebase.google.com/docs/firestore/quickstart#node.js).
2. Convert the body of the blog from HTML to markdown using [Turndown](https://github.com/domchristie/turndown). Nuxt Content also supports other formats such as JSON, YAML, XML and CSV. For writing, I feel that markdown is the most conducive format.
3. Add any extra variables to the [Front Matter](https://content.nuxtjs.org/writing#front-matter) of the content. My blog has properties such as `tags` and `imageUrl`, which stores an article's tags and cover image. The front matter allows us to define the extra variables of a content. 
4. Save the content into the `content` directory with the article's slug as it's filename.

## Retrieving content

I used to retrieve data from Cloud Firestore using code like this:

```js[/src/pages/blog/_id/index.vue]
async asyncData ({ app, params, error }) {
  const db = app.$firebase.firestore()

  try {
    // Get the individual article.
    const documentSnapshot = await db.collection('blogs').doc(params.id).get()

    if (!documentSnapshot.exists) {
      error({ statusCode: 404, message: 'Blog not found' })
      return
    }

    // Get the previous and next articles.
    const promise1 = db
      .collection('blogs')
      .where('published', '==', true)
      .orderBy('created', 'desc')
      .limit(1)
      .startAfter(documentSnapshot)
      .get()

    const promise2 = db
      .collection('blogs')
      .where('published', '==', true)
      .orderBy('created', 'asc')
      .limit(1)
      .startAfter(documentSnapshot)
      .get()

    const prevNext = await Promise.all([promise1, promise2])
      .then((querySnapshots) => {
        const docs = []
        for (const querySnapshot of querySnapshots) {
          if (querySnapshot.empty) {
            docs.push(null)
          } else {
            const doc = querySnapshot.docs[0]
            docs.push({
              id: doc.id,
              title: doc.get('title')
            })
          }
        }
        return docs
      })

    return {
      blog: {
        id: documentSnapshot.id,
        ...documentSnapshot.data()
      },
      prev: prevNext[0],
      next: prevNext[1]
    }
  } catch (e) {
    // eslint-disable-next-line no-console
    console.error(e)
    error({ statusCode: 404, message: 'Blog not found' })
  }
}
```

With Nuxt Content, that becomes this:

```js[/pages/blog/_slug.vue]
async asyncData ({
  $content,
  params
}) {
  // Get the individual article.
  const blog = await $content('blog', params.slug).fetch()

  // Get the previous and next articles.
  const [prev, next] = await $content('blog')
    .only(['title', 'slug', 'createdAt'])
    .sortBy('createdAt', 'asc')
    .surround(params.slug)
    .fetch()

  return {
    blog,
    prev,
    next
  }
}
```

As you can see, the logic is pretty much the same so refactoring my code wasn't a struggle at all.

## Publishing content

Previously, my blog had an admin interface. Whenever I feel like writing, I just log into it and [write content in a rich-text editor](/blog/writing-rich-text-content-firestore-tiptap-nuxt) and publish it.

With a static website, it's different. I create a file in the `content` directory and write on it. Once I feel like publishing it, I just commit it and push it to GitHub. The website is then automatically generated and deployed through GitHub Actions.

## Closing thoughts

I'm definitely happy with the outcome. The process of refactoring was painless, very exciting and, in some ways, refreshing. I also do feel a lot of improvements with this new version in terms of its performance and maintainability. However, there are some downsides to it, which for me weren't really important.

As was and will always be, the [source code for this blog](https://github.com/angheloko/donlalicon-static) is open source and can be accessed [here](https://github.com/angheloko/donlalicon-static).

## References

- [Script to migrate data from Cloud Firestore to Nuxt Content](https://gist.github.com/angheloko/7bbfff17c2b74e7b61d3cb048c4ccf6b)
- [Old dynamic site built with Nuxt and Cloud Firestore](https://github.com/angheloko/donlalicon)
- [New static site built with Nuxt and Nuxt Content](https://github.com/angheloko/donlalicon-static)
