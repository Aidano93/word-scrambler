<template>
  <Scrambler />
</template>

<script>
import Scrambler from './components/Scrambler.vue'

export default {
  name: 'App',
  components: {
    Scrambler
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
