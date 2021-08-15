<template>
  <div
    class="canvas-container transparency-checkerboard"
    @mousemove="mouse"
    @mouseleave="mouse"
    :style="{
      width: displayWidth + 'px',
      height: displayHeight + 'px',
    }"
  >
    <canvas
      v-for="layer in layers"
      :key="layer.id"
      :ref="el => setCanvasRef(el, layer.id)"
      :width="displayWidth"
      :height="displayHeight"
      :style="{
        zIndex: layer.index,
      }"
    />
  </div>
  <button @click="preview = true">Preview</button>
  <Layers :layers="layers" v-model="currentLayer" @add="newLayer" />
  <Preview :layers="layers" v-if="preview" @close="preview = false" />
</template>

<script>
import Layers from './Layers.vue'
import Preview from './Preview.vue'
import { width, height } from '../constants'

const white = '#b1afa8'
const black = '#312f28'

let lastId = 0

export default {
  name: 'Draw',
  components: {
    Layers,
    Preview,
  },
  emits: ['move', 'scale'],
  data: () => ({
    currentLayer: null,
    layers: {},
    canvases: {},
    preview: false,
    width,
    height,
    scale: 1,
  }),
  computed: {
    displayWidth() {
      return this.width * this.scale
    },
    displayHeight() {
      return this.height * this.scale
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
        console.warn(`Can't find canvas for layer ${id}`)
        return
      }

      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      const { data } = this.layers[id]
      if (data) {
        if (this.scale > 1) {
          const img = new Image()
          img.onload = () => {
            ctx.imageSmoothingEnabled = false
            ctx.scale(this.scale, this.scale)
            ctx.drawImage(img, 0, 0)
            ctx.setTransform(1, 0, 0, 1, 0, 0)
          }

          const canvas2 = document.createElement('canvas')
          canvas2.width = width
          canvas2.height = height
          const ctx2 = canvas2.getContext('2d')
          ctx2.putImageData(data, 0, 0)
          img.src = canvas2.toDataURL()
        } else {
          ctx.scale(1, 1)
          ctx.putImageData(data, 0, 0)
        }
      }
    },
    newLayer() {
      lastId += 1
      this.layers[lastId] = {
        id: lastId,
        index: Object.keys(this.layers).length,
        name: `Layer ${lastId}`,
        visible: true,
        data: new ImageData(width, height),
      }
      this.currentLayer = lastId
      return lastId
    },
    clear(id) {
      const data = new ImageData(width, height)
      return this.layers[id].data = data
    },
    draw(id, callback) {
      const canvas = document.createElement('canvas')
      canvas.width = width
      canvas.height = height
      const ctx = canvas.getContext('2d')
      if (this.layers[id].data) {
        ctx.putImageData(this.layers[id].data, 0, 0)
      }
      callback(ctx)
      this.layers[id].data = ctx.getImageData(0, 0, width, height)
    },
    rect(id, x, y, w, h, color) {
      this.draw(id, ctx => {
        ctx.fillStyle = color
        ctx.fillRect(x, y, w, h)
      })
    },
    outlineRect(id, x, y, w, h, color, lineWidth=1) {
      this.draw(id, ctx => {
        ctx.strokeStyle = color
        ctx.lineWidth = lineWidth
        ctx.strokeRect(x, y, w, h)
      })
    },
    mouse(e) {
      if (e.type === 'mousemove') {
        const x = Math.floor(e.offsetX / this.scale)
        const y = Math.floor(e.offsetY / this.scale)
        this.$emit('move', x, y)
        if (e.buttons === 1) {
          this.rect(this.currentLayer, x, y, 1, 1, white)
        }
      } else {
        this.$emit('move', null, null)
      }
    },
    resize() {
      this.scale = Math.floor((window.innerWidth - 32) / width)
      this.$emit('scale', this.scale)
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
  mounted() {
    const id = this.newLayer()
    this.$nextTick(() => {
      this.rect(id, 10, 10, 20, 20, white)
      this.rect(id, 20, 20, 20, 20, black)
      this.outlineRect(id, 30.5, 30.5, 20, 20, white, 1)
    })
    window.addEventListener('resize', this.resize)
    this.resize()
  },
}
</script>

<style scoped>
.canvas-container {
  position: relative;
  margin: 1rem;
}
.canvas-container > canvas {
  position: absolute;
}
</style>
