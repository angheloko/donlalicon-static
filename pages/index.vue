<template>
  <div>
    <NuxtLink
      v-for="blog of blogs"
      :key="blog.slug"
      :to="{ name: 'blog-slug', params: { slug: blog.slug } }"
      class="block mb-8 bg-gray-100 border border-gray-200 shadow-sm rounded no-underline text-gray-800 transition-colors duration-300 ease-linear hover:text-gray-800 hover:bg-gray-200 sm:flex sm:flex-row-reverse sm:p-4"
    >
      <div v-if="blog.cover" class="sm:flex-shrink-0 sm:w-1/4 sm:ml-4">
        <img
          :src="blog.cover.thumb"
          :alt="blog.cover.alt"
          class="h-auto rounded-t object-cover sm:rounded-none"
        >
      </div>
      <div class="flex-grow flex flex-col p-4 sm:p-0">
        <h2 class="font-serif">
          {{ blog.title }}
        </h2>
        <h3 class="flex-grow my-4">
          {{ blog.subtitle }}
        </h3>
        <div v-if="blog.tags" class="uppercase text-xs font-bold text-gray-600 mb-4">
          <div v-for="tag of blog.tags.slice(0).sort()" :key="tag" class="mr-2 inline-block">
            {{ tag }}
          </div>
        </div>
        <div class="text-sm text-gray-600">
          {{ blog.createdAt | toDate }}
        </div>
      </div>
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
      .only(['title', 'subtitle', 'slug', 'cover', 'createdAt', 'tags'])
      .sortBy('createdAt', 'desc')
      .fetch()

    return {
      blogs
    }
  }
}
</script>
