<template>
  <Wrap :page="page">
    <article class="post">
      <div class="post-content">
        <slot name="default"></slot>
      </div>
    </article>
  </Wrap>
</template>

<script>
import Wrap from '../components/Wrap.vue'

export default {
  components: {
    Wrap
  },

  props: ['page']
}
</script>
