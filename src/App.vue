<template>
  <StringMatchingVisualization msg="Welcome to Your Vue.js App"/>
</template>

<script>
import StringMatchingVisualization from './components/StringMatchingVisualization.vue'

export default {
  name: 'App',
  components: {
    StringMatchingVisualization
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
