<template>
  <div class="relative">
    <div>
      <input
        v-model="searchQuery"
        type="search"
        autocomplete="off"
        placeholder="Search"
        class="border border-transparent text-gray-700 placeholder-gray-600 rounded-lg bg-gray-200 py-2 pr-4 pl-10 block w-full appearance-none leading-normal outline-none transition-colors ease-linear duration-300 focus:bg-white focus:border-gray-300 focus:shadow"
        @focus="showResults = true"
      >
      <ul v-if="articles.length && showResults" class="absolute w-full left-0 mt-1 p-0 z-50 list-none bg-white rounded-lg shadow">
        <li v-for="article of articles" :key="article.slug" class="px-4 py-2">
          <NuxtLink
            :to="{ name: 'blog-slug', params: { slug: article.slug } }"
            class="no-underline block"
            @click.native="showResults = false"
          >
            {{ article.title }}
          </NuxtLink>
        </li>
      </ul>
    </div>
    <div class="pointer-events-none absolute inset-y-0 left-0 pl-4 flex items-center">
      <svg class="fill-current pointer-events-none text-gray-600 w-4 h-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
        <path d="M12.9 14.32a8 8 0 1 1 1.41-1.41l5.35 5.33-1.42 1.42-5.33-5.34zM8 14A6 6 0 1 0 8 2a6 6 0 0 0 0 12z" />
      </svg>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      searchQuery: '',
      articles: [],
      showResults: false,
      onClickEvent: (event) => {
        if (!this.$el.contains(event.target) && this.$el !== event.target) {
          this.showResults = false
        }
      }
    }
  },
  watch: {
    async searchQuery (searchQuery) {
      if (!searchQuery) {
        this.articles = []
        return
      }
      this.articles = await this.$content('blog')
        .limit(6)
        .search(searchQuery)
        .fetch()
    }
  },
  mounted () {
    document.addEventListener('click', this.onClickEvent)
  },
  beforeDestroy () {
    document.removeEventListener('click', this.onClickEvent)
  }
}
</script>
