<template>
  <div class="layers">
    <ol>
      <li
        v-for="layer in sortedLayers"
        :key="layer.id"
        class="flex items-center"
        @click="setLayer(layer.id)"
      >
        <canvas
          :ref="el => setCanvasRef(el, layer.id)"
          :width="width"
          :height="height"
          class="layer-preview transparency-checkerboard"
        />
        <div>
          {{ layer.name }}
          <span class="current" v-if="layer.id === modelValue">
            (current)
          </span>
        </div>
      </li>
    </ol>
    <button @click="$emit('add')">New layer</button>
  </div>
</template>

<script>
import { sortBy } from 'lodash'
import { width, height } from '../constants'

export default {
  props: ['layers', 'modelValue'],
  emits: ['update:modelValue', 'add'],
  data: () => ({
    canvases: {},
    width,
    height,
  }),
  computed: {
    sortedLayers() {
      return sortBy(this.layers, layer => layer.index)
    },
  },
  methods: {
    setCanvasRef(el, id) {
      if (el) {
        this.canvases[id] = el
      }
    },
    render(id) {
      const canvas = this.canvases[id]
      if (!canvas) {
        return
      }
      const ctx2 = canvas.getContext('2d')
      ctx2.clearRect(0, 0, canvas.width, canvas.height)
      const { data } = this.$props.layers[id]
      if (data) {
        ctx2.putImageData(data, 0, 0)
      }
    },
    setLayer(id) {
      this.$emit('update:modelValue', id)
    },
  },
  watch: {
    layers: {
      handler(newVal) {
        Object.keys(newVal).forEach(id => {
          this.render(id)
        })
      },
      deep: true,
    },
  },
  beforeUpdate() {
    this.canvases = {}
  },
}
</script>

<style scoped>
.layers {
  margin: 1rem;
}
.layers ol {
  list-style: none;
  margin: 0;
  padding: 0;
}
.layers li {
  margin-bottom: 2px;
}
.layer-preview {
  width: 60px;
  height: 36px;
  margin-right: 0.5rem;
}
.layers .current {
  color: var(--purple);
}
@media (prefers-color-scheme: dark) {
  .layers .current {
    color: var(--yellow);
  }
}
</style>
