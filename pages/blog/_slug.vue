<template>
  <article>
    <div v-if="blog.tags" class="uppercase text-xs font-bold text-gray-600">
      <div v-for="tag of blog.tags.slice(0).sort()" :key="tag" class="mr-2 inline-block">
        {{ tag }}
      </div>
    </div>
    <h1>
      {{ blog.title }}
    </h1>
    <h2 class="text-gray-700">
      {{ blog.lead }}
    </h2>
    <div class="text-sm text-gray-600">
      {{ blog.createdAt | toDate }}
    </div>
    <figure v-if="blog.cover" class="my-4">
      <img :src="blog.cover.image" :alt="blog.cover.alt">
      <figcaption v-html="blog.cover.caption" />
    </figure>
    <nav>
      <ul class="px-4 my-4">
        <li v-for="link of blog.toc" :key="link.id" class="pb-2 pl-1" :class="{ 'ml-5': link.depth === 3 }">
          <NuxtLink :to="`#${link.id}`" class="no-underline">
            {{ link.text }}
          </NuxtLink>
        </li>
      </ul>
    </nav>
    <nuxt-content :document="blog" class="my-8" />
  </article>
</template>
<script>
export default {
  async asyncData ({
    $content,
    params
  }) {
    const blog = await $content('blog', params.slug).fetch()

    return { blog }
  },
  head () {
    const url = `https://donlalicon.dev/blog/${this.blog.slug}`

    const dateCreated = new Date(this.blog.createdAt)
    const dateChanged = new Date(this.blog.updatedAt)

    const structuredData = {
      '@type': 'Article',
      datePublished: dateCreated.toISOString(),
      dateModified: dateChanged.toISOString(),
      headline: this.blog.title,
      image: this.blog.cover.image
    }

    const head = {
      title: this.blog.title,
      link: [
        {
          rel: 'canonical',
          href: url
        }
      ],
      script: [
        {
          type: 'application/ld+json',
          json: structuredData
        }
      ],
      meta: [
        {
          hid: 'og:url',
          name: 'og:url',
          property: 'og:url',
          content: url
        },
        {
          hid: 'og:title',
          name: 'og:title',
          property: 'og:title',
          content: `${this.blog.title} - donlalicon.dev`
        }
      ]
    }

    let description = this.blog.title
    if (this.blog.description) {
      description = this.blog.description
    }

    head.meta.push(
      {
        hid: 'description',
        name: 'description',
        content: description
      },
      {
        hid: 'og:description',
        name: 'og:description',
        property: 'og:description',
        content: description
      }
    )

    if (this.blog.imageUrl) {
      head.meta.push({
        hid: 'og:image',
        name: 'og:image',
        property: 'og:image',
        content: this.blog.imageUrl
      })
    }

    return head
  }
}
</script>
