<template>
  <article>
    <h1>
      {{ blog.title }}
    </h1>
    <h2 class="text-gray-700">
      {{ blog.lead }}
    </h2>
    <div class="text-sm text-gray-600">
      {{ blog.createdAt | toDate }}
    </div>
    <figure>
      <img :src="blog.cover.image" :alt="blog.cover.alt">
      <figcaption>
        {{ blog.cover.caption }}
      </figcaption>
    </figure>
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
  }
}
</script>
