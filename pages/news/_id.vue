<template>
  <div class="container">
    <header class="header d-flex flex-column">
      <logo />

      <div class="links">
        <nuxt-link to="/" class="button--green">
          Home
        </nuxt-link>

        <nuxt-link to="/news/" class="button--green">
          News
        </nuxt-link>
      </div>

      <h1 class="title" v-html="currentPage.title.rendered" />

      <div class="d-flex justify-content-between align-items-center">
        <button
          class="button--grey"
          :disabled="pageNumber === 0"
          @click="prevPage"
        >
          Prev
        </button>

        <span>{{ pageNumber + 1 }}/{{ pageCount }}</span>

        <button
          class="button--grey"
          :disabled="pageNumber >= pageCount - 1"
          @click="nextPage"
        >
          Next
        </button>
      </div>
    </header>

    <hr />

    <section class="row">
      <div class="card-columns">
        <div v-for="post in paginatedData" :key="post.id" class="card">
          <nuxt-link class="card-link" :to="`/news/${post.id}/`">
            <img
              v-if="post.featured_media_url"
              class="card-img-top"
              :src="post.featured_media_url"
              alt="Card image cap"
            />

            <div class="card-body">
              <h5 class="card-title" v-html="post.title.rendered" />

              <div class="card-text" v-html="post.excerpt.rendered" />
            </div>
          </nuxt-link>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'

export default {
  name: 'Post',
  components: {
    Logo
  },
  data() {
    return {
      pageNumber: 0,
      perPage: 3
    }
  },
  computed: {
    pageCount() {
      const l = this.relatedPosts.length
      const s = this.perPage

      return Math.ceil(l / s)
    },
    paginatedData() {
      const start = this.pageNumber * this.perPage
      const end = start + this.perPage

      return this.relatedPosts.slice(start, end)
    }
  },
  asyncData({ $axios, params }) {
    const offset = Math.floor(Math.random() * 100)
    return Promise.all([
      // Fetch more posts
      $axios.$get(
        `https://veja.abril.com.br/wp-json/wp/v2/posts/?per_page=9&offset=${offset}`
      ),
      // Fetch current post
      $axios.$get(`https://veja.abril.com.br/wp-json/wp/v2/posts/${params.id}`)
    ]).then(([relatedPosts, currentPage]) => {
      return {
        relatedPosts: relatedPosts,
        currentPage: currentPage
      }
    })
  },
  methods: {
    changePage(event) {
      this.pageNumber = event.target.value - 1
    },
    nextPage() {
      this.pageNumber++
    },
    prevPage() {
      this.pageNumber--
    }
  }
}
</script>

<style scoped>
.header {
  align-items: center;
  justify-content: center;
  margin: 2rem auto;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
