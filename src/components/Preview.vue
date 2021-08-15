<template>
  <div class="preview-container">
    <div class="playdate">
      <div class="playdate-bezel">
        <div class="canvas-container">
          <canvas
            v-for="layer in layers"
            :key="layer.id"
            :ref="el => setCanvasRef(el, layer.id)"
            :width="width"
            :height="height"
            :style="{
              zIndex: layer.index,
            }"
          />
        </div>
      </div>
    </div>
    <button class="close" @click="$emit('close')">
      Close
    </button>
  </div>
</template>

<script>
import { width, height } from '../constants'

export default {
  name: 'Preview',
  props: ['layers'],
  emits: ['close'],
  data: () => ({
    canvases: {},
    width,
    height,
  }),
  methods: {
    setCanvasRef (el, id) {
      if (el) {
        this.canvases[id] = el;
      }
    },
    render(id) {
      const canvas = this.canvases[id]
      if (!canvas) {
        console.warn(`Can't find canvas for layer ${id}`)
        return
      }

      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      const { data } = this.layers[id]
      if (data) {
        ctx.putImageData(data, 0, 0)
      }
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
.preview-container {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 20;
}
.playdate {
  font-size: 16px;
  display: inline-block;
  margin: 1em;
  background-color: var(--device-yellow);
  border-radius: 0.5em;
  padding-top: 0.5em;
  padding-left: 0.5em;
  padding-right: 2em;
  padding-bottom: 14em;
}
.playdate-bezel {
  padding: 0.5rem;
  background-color: var(--psd-darkest-gray);
  border-radius: 0.25rem;
}
.canvas-container {
  position: relative;
  width: 25em;
  height: 15em;
  background-color: var(--screen-black);
}
.canvas-container > canvas {
  position: absolute;
}
.close {
  position: absolute;
  top: 0.5rem;
  right: 0.75rem;
}
</style>
