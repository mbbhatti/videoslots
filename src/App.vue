<template>
  <div id="app">
    <ImageGallery msg="Image Gallery"/>
  </div>
</template>

<script>
import ImageGallery from './components/ImageGallery.vue'

export default {
  name: 'app',
  components: {
    ImageGallery
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
</style>
