<template>
  <article>
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
