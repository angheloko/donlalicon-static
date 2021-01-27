<template>
  <div>
    <NuxtLink
      v-for="blog of blogs"
      :key="blog.slug"
      :to="{ name: 'blog-slug', params: { slug: blog.slug } }"
      class="flex mb-4 bg-gray-100 p-4 rounded no-underline text-gray-800 transition-colors duration-300 ease-linear hover:text-gray-800 hover:bg-gray-200"
    >
      <div class="flex-grow flex flex-col">
        <h2 class="font-serif">
          {{ blog.title }}
        </h2>
        <h3 class="flex-grow">
          {{ blog.subtitle }}
        </h3>
        <div class="text-sm text-gray-600">
          {{ blog.createdAt | toDate }}
        </div>
      </div>
      <img :src="blog.cover.thumb" :alt="blog.cover.alt" class="flex-shrink w-1/4 object-cover">
    </NuxtLink>
  </div>
</template>
<script>
export default {
  async asyncData ({
    $content,
    params
  }) {
    const blogs = await $content('blog', params.slug)
      .only(['title', 'subtitle', 'slug', 'cover', 'createdAt'])
      .sortBy('createdAt', 'asc')
      .fetch()

    return {
      blogs
    }
  }
}
</script>
