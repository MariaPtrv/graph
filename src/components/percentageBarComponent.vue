<template>
  <div class='layout'>
    <div class="fill" v-bind:style="{width: computedWidth + '%'}"></div>
  </div>
</template>

<script>
export default {
  name: "percentageBarComponent"
}
</script>

<script setup>
import {computed, defineProps} from "vue";

const props = defineProps({
  threshold: String
});

const computedWidth = computed(() => props.threshold * 100);
console.log(computedWidth.value)
</script>

<style scoped>
.layout {
  width: 100%;
  height: 100%;
  border-radius: 5px;
  background: grey;
}

.fill {
  background: green;
  border-radius: 5px 0 0 5px;
  height: 100%;
}
</style>