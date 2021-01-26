<template>
  <article class="blog">
    <h1>
      {{ blog.title }}
    </h1>
    <h2>
      {{ blog.lead }}
    </h2>
    <div class="blog__date">
      {{ blog.createdAt | toDate }}
    </div>
    <figure>
      <img :src="blog.cover.image" :alt="blog.cover.alt">
      <figcaption>
        {{ blog.cover.caption }}
      </figcaption>
    </figure>
    <nuxt-content :document="blog" class="blog__content" />
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
