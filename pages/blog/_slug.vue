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
