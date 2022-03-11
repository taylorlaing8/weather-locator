<template>
  <MapLocator/>
</template>

<script>
import MapLocator from './components/MapLocator'

export default {
  name: 'App',
  components: {
    MapLocator
  }
}
</script>

<style lang="scss">
@import "~bootstrap/scss/bootstrap";

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
