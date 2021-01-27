<template>
  <div
    :class="{ 'w-full': isFocused }"
    class="absolute right-0 px-4 sm:relative sm:px-0 sm:w-auto sm:relative"
  >
    <div class="w-full">
      <input
        v-model="searchQuery"
        :class="{ 'search-box--focus': isFocused }"
        type="search"
        autocomplete="off"
        placeholder="Search"
        class="search-box border border-transparent text-gray-700 placeholder-gray-600 rounded py-2 px-5 block w-0 appearance-none leading-normal outline-none sm:w-full sm:pl-10 sm:pr-4 sm:bg-gray-200"
        @focus="isFocused = true"
      >
      <div class="px-4 relative">
        <ul
          v-if="articles.length && isFocused"
          class="search-results absolute w-full left-0 mt-1 px-0 py-2 z-50 list-none bg-white rounded shadow-lg border border-gray-200"
        >
          <li v-for="article of articles" :key="article.slug" class="px-4 py-2 hover:bg-gray-200">
            <NuxtLink
              :to="{ name: 'blog-slug', params: { slug: article.slug } }"
              class="no-underline block"
              @click.native="isFocused = false"
            >
              {{ article.title }}
            </NuxtLink>
          </li>
        </ul>
      </div>
    </div>
    <div class="pointer-events-none absolute inset-y-0 pl-3 flex items-center">
      <svg
        class="search-icon fill-current pointer-events-none text-gray-600 w-5 h-5"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 20 20"
      >
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
      isFocused: false,
      onClickEvent: (event) => {
        if (!this.$el.contains(event.target) && this.$el !== event.target) {
          this.isFocused = false
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
        .only(['title', 'subtitle', 'slug'])
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
.search-box--focus,
.search-box:focus {
  @apply w-full pl-10 pr-4 bg-white shadow border border-gray-200;
}

.search-results {
  top: -5px;
}
</style>
