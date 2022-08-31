<template>
  <div class="field">
    <percentageBarComponent :threshold=test.threshold
                            :min=test.min
                            :max=test.max
                            :unit=test.unit
                            :precision=test.precision
    ></percentageBarComponent>
  </div>
</template>

<script>
import percentageBarComponent from "@/components/percentageBarComponent";

export default {
  name: 'App',
  components:
  percentageBarComponent

}
</script>

<script setup>
// import {ref} from 'vue';

const test = {
  threshold: 0.5,
  min: 14,
  max: 24,// день в часах
  unit: 'ч',
  precision: 2, // 1 знак после запятой
}


// const data = ref({
//   states:['ожидание', 'подача колодок', 'подача гармошки','выгрузка пасссажиров', 'выгрузка багажа', 'уборка самолёта'],
//   adjacencies:
//       [
//         [false, false, false, true, false, false],
//         [false, false, false, false, true, false],
//         [false, false, false, true, false, false],
//         [false, false, false, false, true, false],
//         [true, false, false, false, false, false],
//         [true, false, false, false, false, false]
//       ]
// })
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

.field {
  height: 50px;
  width: 80%;
}
</style>
