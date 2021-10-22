<template>
  <div id="app">
    <ScopeSlider v-model="data1" :min="0" :max="10" :barColors="barColorSet"/>
    <div style="margin-left:100px;"/>
    <ScopeSlider v-model="data2" :vertical="true" :min="0" :max="10" />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from '@vue/composition-api'
import ScopeSlider from './lib/index.vue'

export default defineComponent({
  components: {
    ScopeSlider
  },
  setup() {
    const data1 = [2, 5, 8];
    const data2 = ref(0);

    const barColorSet = [
      '#409eff', '#ffb340', '#ff4049', '#409eff'
    ]

    return {
      data1,
      data2,
      barColorSet
    }
  }
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

<style lang="scss">
#app {
  display: flex;
  justify-content: center;
}
</style>
