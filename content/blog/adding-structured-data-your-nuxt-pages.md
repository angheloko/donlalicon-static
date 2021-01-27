---
title: "Adding Structured Data to Your Nuxt Pages"
subtitle: "Helping Google Search by adding structured data into your Nuxt pages"
lead: "Helping Google Search by adding structured data into your Nuxt pages"
description: "Helping Google Search by adding structured data into your Nuxt pages"
createdAt: 2020-01-12T03:28:34.215Z
updatedAt: 2020-01-12T04:08:14.685Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/adding-structured-data-your-nuxt-pages%2Fsai-kiran-anagani-5Ntkpxqt54Y-unsplash.jpg?alt=media&token=7e7f080b-fabd-4b42-a3db-c5d93735ddb6"
  alt: "Adding Structured Data to Your Nuxt Pages cover image"
  caption: "Photo by Sai Kiran Anagani on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/adding-structured-data-your-nuxt-pages%2Fsai-kiran-anagani-5Ntkpxqt54Y-unsplash_200x200.jpg?alt=media&token=7e7f080b-fabd-4b42-a3db-c5d93735ddb6"
tags: 
  - Structured data
  - Google Search
  - Nuxt
  - Vue Meta
---
<p><a href="https://developers.google.com/search/docs/guides/intro-structured-data" rel="noopener noreferrer nofollow">Structured data</a> helps Google Search understand the content of your pages better. It's a standardised format for providing information about a page and classifying the page content.</p><p>You can add structured data by using the <a href="https://json-ld.org/" rel="noopener noreferrer nofollow">JSON-LD</a> format. Below is an example of a structured data for a simple article:</p><pre><code>&lt;script type="application/ld+json"&gt;
{
  "@type": "Article",
  "headline": "Article headline",
  "image": "https://example.com/photos/1x1/photo.jpg",
  "datePublished": "2015-02-05T08:00:00+08:00",
  "dateModified": "2015-02-05T09:20:00+08:00"
}
&lt;/script&gt;</code></pre><p>There are other <a href="https://developers.google.com/search/docs/data-types/article#amp" rel="noopener noreferrer nofollow">properties</a> that you can specify in the structured data depending on the type of content. For instance, a recipe content type could have <code>cookTime</code> and <code>recipeInstructions</code>.</p><h2>Adding structured data via the head method</h2><p>I decided to add the structure data inside the <code>&lt;head&gt;</code> section of the page so naturally I looked into the <a href="https://nuxtjs.org/api/pages-head#the-head-method" rel="noopener noreferrer nofollow">head method</a>, which uses <a href="https://github.com/nuxt/vue-meta" rel="noopener noreferrer nofollow">vue-meta</a>.</p><p>Since I was adding a script tag, I ended up with the following (trimmed for brevity):</p><pre><code>&lt;script&gt;
export default {
  head () {
    const dateCreated = new Date(this.blog.created.seconds * 1000)
    const dateChanged = new Date(this.blog.changed.seconds * 1000)

    const structuredData = {
      '@type': 'Article',
      datePublished: dateCreated.toISOString(),
      dateModified: dateChanged.toISOString(),
      headline: this.blog.title,
      image: this.blog.imageUrl
    }

    return {
      script: [
        {
          type: 'application/ld+json',
          json: structuredData
        }
      ]
    }
  }
}
&lt;/script&gt;</code></pre><p>First, we construct the structured data from the blog data and add it into the <a href="https://vue-meta.nuxtjs.org/api/#script" rel="noopener noreferrer nofollow">script property</a>. Luckily, Vue Meta supports adding <a href="https://vue-meta.nuxtjs.org/api/#add-json-data" rel="noopener noreferrer nofollow">JSON objects into script tags</a> since version 2.1.</p><p>You can see the full <a href="https://github.com/angheloko/donlalicon/blob/master/pages/blog/_id/index.vue" rel="noopener noreferrer nofollow">source code</a> from the repo.</p><h2>Pre-Vue Meta 2.1</h2><p>If for some reason you are stuck with an older version of Vue Meta, you can still inject custom code into the script tag inside the head of your page with the use of <code>__dangerouslyDisableSanitizers</code> or <code>__dangerouslyDisableSanitizersByTagID</code>.</p><p>Example of how the earlier code will be adjusted:</p><pre><code>return {
  __dangerouslyDisableSanitizers: ['script'],
  script: [
    {
      type: 'application/ld+json',
      innerHTML: JSON.stringify(structuredData)
    }
  ]
}</code></pre><p>Base from the name of these properties, it goes without saying that their use is <strong>highly discouraged</strong>.</p><h2>Closing</h2><p>You can add structured data in <a href="https://developers.google.com/search/docs/guides/intro-structured-data#structured-data-format" rel="noopener noreferrer nofollow">other formats</a> too, although, Google recommends <code>JSON+LD</code>.</p><p>You can also add the structured data in the page's body instead of head. You can even inject it dynamically, according to the documentation. My tests say otherwise but your results may vary.</p><p>I opted to initialise the blog data in the <code>asyncData</code> method and insert the structured data into the head. Afterall, doing both is straightforward with Nuxt and Vue Meta.</p>