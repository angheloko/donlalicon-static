<template>
  <div
    :class="{ 'w-full': showResults }"
    class="absolute right-0 px-4 sm:relative sm:pr-0 sm:w-1/3 sm:relative"
  >
    <div>
      <input
        v-model="searchQuery"
        :class="{ 'search-box--expanded': showResults }"
        type="search"
        autocomplete="off"
        placeholder="Search"
        class="search-box border border-transparent text-gray-700 placeholder-gray-600 rounded py-2 px-6 block w-0 appearance-none leading-normal outline-none sm:w-full sm:pl-10 sm:pr-4 sm:bg-gray-200"
        @focus="showResults = true"
      >
      <div class="px-4 relative">
        <ul v-if="articles.length && showResults" class="absolute w-full left-0 mt-1 px-0 py-2 z-50 list-none bg-white rounded shadow-lg border border-gray-200">
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
    </div>
    <div class="pointer-events-none absolute inset-y-0 pl-4 flex items-center">
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
<style scoped>
.search-box--expanded,
.search-box:focus {
  @apply w-full pl-10 pr-4 bg-white shadow border border-gray-200;
}
</style>
