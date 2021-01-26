<template>
  <article>
    <h1>
      {{ blog.title }}
    </h1>
    <h2>
      {{ blog.lead }}
    </h2>
    <div>
      {{ blog.createdAt | toDate }}
    </div>
    <figure>
      <img :src="blog.cover.image" :alt="blog.cover.alt">
      <figcaption>{{ blog.cover.caption }}</figcaption>
    </figure>
    <nuxt-content :document="blog" />
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
  }
}
</script>
