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
  states:['ожидание', 'подача колодок', 'подача гармошки'],
  adjacencies:
      [
        [false, true, true],
        [false, false, true],
        [false, true,false]
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
