<template>
  <Wrap :page="page">
    <article class="post">
      <header class="post-header">
        <h1 class="post-title">{{ page.title }}</h1>
        <p class="post-meta">
          <time class="dt-published" :datetime="page.createdAt" itemprop="datePublished">{{ formatDate(page.createdAt) }}</time>
        </p>
      </header>

      <div class="post-content" itemprop="articleBody">
        <slot name="default"/>
      </div>

      <a class="u-url" :href="page.permalink" hidden></a>
    </article>
  </Wrap>
</template>

<script>
import formatDate from '../utils/formatDate'
import Wrap from '../components/Wrap.vue'

export default {
  components: {
    Wrap,
  },

  props: ['page'],

  methods: {
    formatDate
  }
}
</script>

<style lang="scss">

</style>
