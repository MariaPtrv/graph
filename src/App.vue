<template>
  <processVisualization :data=data />
</template>

<script>
import processVisualization from './components/processVisualization.vue'

export default {
  name: 'App',
  components: {
    processVisualization
  }
}
</script>

<script setup>
import {ref} from 'vue';

const data = ref({
  states:['titlt 1',' titlt 2','titlt 3',' titlt 4'],
  adjacencies:
      [
        [false, false, true,  false],
        [false, false, false, false],
        [true,  false, false, true],
        [false, true,  false, false]
      ]
})
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
