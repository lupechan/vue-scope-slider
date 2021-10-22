<template>
  <div class="scope-slider" :class="vertical ? 'vertical' : 'horizontal'">
    <div class="scope-slider-axis">
        <Tap
          v-for="(value, index) in scopeValues"
          :key="`tap_${index}`"
          v-model="scopeValues[index]"
          :max="index === scopeValues.length - 1 ? max : scopeValues[index + 1]"
          :min="index === 0 ? min : scopeValues[index - 1]"
          @input="getHandleInput(index, $event)"
        />
        <div
          v-for="(styles, index) in scopeStyles"
          :key="`bar_${index}`"
          :style="styles"
          class="scope-slider-bar"
        />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, provide, ref, computed, set } from '@vue/composition-api'
import '../assets/style.scss'
import Tap from './tap.vue'

export default defineComponent({
  name: 'ScopeSlider',
  components: { Tap },
  props: {
    value: {
      type: [Number, Array],
      default: 0,
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    vertical: {
      type: Boolean,
      default: false,
    },
    step: {
      type: Number,
      default: 1,
    },
    barColors: {
      type: [Array, Function],
      default: () => ['#409eff']
    }
  },
  setup(props, context) {
    const sliderSize = ref(300)
    const scopeValues = ref(
      Array.isArray(props.value)
        ? (props.value as number[])
        : [props.value]
    )
    const scopeStyles = computed(() => {
      const scope = [props.min, ...scopeValues.value, props.max]
      const styles = []
      const total = props.max - props.min
      const poiKeys = props.vertical ? ['bottom', 'height'] : ['left', 'width']
      for (let i = 0; i < scope.length - 1; i++) {
        const current = `${(scope[i] / total) * 100}%`
        const length = `${((scope[i + 1] - scope[i]) / total) * 100}%`
        styles.push({
          [poiKeys[0]]: current,
          [poiKeys[1]]: length,
          backgroundColor: props.barColors[i] || ''
        })
      }
      return styles
    })

    provide('options', props)
    provide('sliderSize', sliderSize)

    const getHandleInput = (index: number, val: number) => {
      Array.isArray(props.value) ?
        set(props.value, index, val)
        : context.emit('input', val)
    }

    return {
      scopeValues,
      getHandleInput,
      scopeStyles
    }
  },
})
</script>
