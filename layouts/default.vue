<template>
  <div>
    <HeaderTop />
    <Nuxt />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'nuxt-composition-api'
import HeaderTop from '~/components/HeaderTop.vue'

export default defineComponent({
  components: { HeaderTop },
  setup() {},
})
</script>

<style>
html {
  font-family: 'Source Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI',
    Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 16px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
  color: #333;
}

body {
  padding: 20px 30px;
}
</style>
