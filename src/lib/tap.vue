<template>
  <div class="scope-slider-tap-wrap" :class="wrapClasses" :style="styles">
    <div
      class="scope-slider-tap"
      :class="wrapClasses"
      @mousedown="onButtonDown"
      @touchstart="onButtonDown"
    />
    <div class="scope-slider-tooltip">{{ tip }}</div>
  </div>
  
</template>

<script lang="ts">
import { defineComponent, ref, computed, inject } from '@vue/composition-api'

export default defineComponent({
  props: {
    value: {
      type: Number,
      default: 0,
    },
    min: {
      type: Number,
      required: true,
    },
    max: {
      type: Number,
      required: true,
    },
    disabled: {
      type: Boolean,
      default: false,
    }
  },
  setup(props, context) {
    const dragging = ref(false)
    const isClick = ref(false)
    const startX = ref(0)
    const currentX = ref(0)
    const startY = ref(0)
    const currentY = ref(0)
    const startPosition = ref(0)
    const newPosition = ref(0)
    const options = inject('options', {} as any)
    const sliderSize = inject('sliderSize', ref(100))

    const currentPosition = computed(() => {
      const { min, max } = options
      return `${((props.value - min) / (max - min)) * 100}%`
    })

    const styles = computed(() => {
      return options.vertical
        ? { bottom: currentPosition.value }
        : { left: currentPosition.value }
    })

    const tip = computed(() => {
      if (typeof options.formatTooltip === 'function') {
        return options.formatTooltip(props.value)
      }
      return props.value
    })

    const wrapClasses = computed(() => {
      return {
        hover: dragging.value
      }
    })

    const setPosition = (position: number) => {
      if (position === null || isNaN(position)) return
      
      const { min, max, step } = options
      const lengthPerStep = 100 / ((max - min) / step)
      const steps = Math.round(position / lengthPerStep)
      let value = steps * lengthPerStep * (max - min) * 0.01 + min
      // value = parseFloat(value.toFixed(this.precision));

      if (value < props.min) {
        value = props.min
      } else if (value > props.max) {
        value = props.max
      }

      context.emit('input', value)
      // this.$nextTick(() => {
      //   this.displayTooltip();
      //   this.$refs.tooltip && this.$refs.tooltip.updatePopper();
      // });
      // if (!dragging && this.value !== this.oldValue) {
      //   this.oldValue = this.value;
      // }
    }
    const onDragStart = (event: MouseEvent | TouchEvent) => {
      dragging.value = true
      isClick.value = true
      if (options.vertical) {
        startY.value =
          event instanceof TouchEvent ? event.touches[0].clientY : event.clientY
      } else {
        startX.value =
          event instanceof TouchEvent ? event.touches[0].clientX : event.clientX
      }
      startPosition.value = parseFloat(currentPosition.value)
      newPosition.value = startPosition.value
    }

    const onDragging = (event: MouseEvent | TouchEvent) => {
      if (!dragging.value) return
      isClick.value = false
      // this.displayTooltip();
      // this.$parent.resetSize();
      let diff = 0
      if (options.vertical) {
        currentY.value =
          event instanceof TouchEvent ? event.touches[0].clientY : event.clientY
        diff = ((startY.value - currentY.value) / sliderSize.value) * 100
      } else {
        currentX.value =
          event instanceof TouchEvent ? event.touches[0].clientX : event.clientX
        diff = ((currentX.value - startX.value) / sliderSize.value) * 100
      }
      newPosition.value = startPosition.value + diff
      setPosition(newPosition.value)
    }

    const onDragEnd = () => {
      if (!dragging.value) return
      setTimeout(() => {
        dragging.value = false
        // this.hideTooltip();
        if (!isClick) {
          setPosition(newPosition.value)
          // this.$parent.emitChange();
        }
      }, 0)
      window.removeEventListener('mousemove', onDragging) 
      window.removeEventListener('touchmove', onDragging)
      window.removeEventListener('mouseup', onDragEnd)
      window.removeEventListener('touchend', onDragEnd)
      window.removeEventListener('contextmenu', onDragEnd)
    }

    const onButtonDown = (event: MouseEvent | TouchEvent) => {
      if (props.disabled) return
      event.preventDefault()
      onDragStart(event)
      window.addEventListener('mousemove', onDragging)
      window.addEventListener('touchmove', onDragging)
      window.addEventListener('mouseup', onDragEnd)
      window.addEventListener('touchend', onDragEnd)
      window.addEventListener('contextmenu', onDragEnd)
    }

    return {
      tip,
      styles,
      wrapClasses,
      onButtonDown,
    }
  },
})
</script>
