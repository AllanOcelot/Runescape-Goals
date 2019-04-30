<template>
  <div id="app">
    <homescreen/>
  </div>
</template>

<script>
import homescreen from './components/homescreen.vue'

export default {
  name: 'app',
  components: {
    homescreen
  }
}
</script>

<style>
body {
    padding: 0;
    margin: 0;
    font-family: 'Montserrat', sans-serif;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
}
</style>
