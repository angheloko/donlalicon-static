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
    <figure class="my-4">
      <img :src="blog.cover.image" :alt="blog.cover.alt">
      <figcaption>
        {{ blog.cover.caption }}
      </figcaption>
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
  }
}
</script>
