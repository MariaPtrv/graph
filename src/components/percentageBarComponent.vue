<template>
  <div class='layout'>
    <div class="fill" :style="{width: computedWidth + '%'}">{{ computedLabel }} {{}}</div>
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
  threshold: Number,
  min: Number,
  max: Number,
  unit: String,
  precision: Number
});

const computedWidth = computed(() => props.threshold * 100);
const computedLabel = computed(() => {
  const value = Number((props.max - props.min) * props.threshold).toFixed(props.precision);
  return `${value} ${props.unit}`;
});

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
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>