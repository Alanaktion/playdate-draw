<template>
  <div class="playdate">
    <div class="playdate-bezel">
      <div class="canvas-container" @mousemove="point" @mouseleave="point">
        <template v-for="(layer, i) in layers" :key="i">
          <canvas
            :ref="el => setCanvasRef(el, i)"
            :width="width"
            :height="height"
            :style="{
              zIndex: i,
            }"
          />
        </template>
      </div>
    </div>
  </div>
  <div class="layers">
    <ol>
      <li
        v-for="(layer, i) in layers"
        :key="i"
        class="flex items-center"
        @click="currentLayer = i"
      >
        <canvas
          :ref="el => setLayerCanvasRef(el, i)"
          :width="width"
          :height="height"
          class="layer-preview"
        />
        <div>
          Layer {{ i + 1 }}
          <span class="current" v-if="i === currentLayer">
            (current)
          </span>
        </div>
      </li>
    </ol>
    <button @click="newLayer">New layer</button>
  </div>
</template>

<script>
const width = 400
const height = 240

const white = '#b1afa8'
const black = '#312f28'

export default {
  name: 'Draw',
  emits: ['move'],
  data: () => ({
    currentLayer: null,
    layers: [],
    canvases: {},
    layerCanvases: {},
    width,
    height,
  }),
  methods: {
    setCanvasRef(el, i) {
      if (el) {
        this.canvases[i] = el
      }
    },
    setLayerCanvasRef(el, i) {
      if (el) {
        this.layerCanvases[i] = el
      }
    },
    render(layer) {
      const canvas = this.canvases[layer]
      if (!canvas) {
        console.warn(`Can't find canvas for layer ${layer}`)
        return
      }
      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      if (this.layers[layer]) {
        ctx.putImageData(this.layers[layer], 0, 0)
      }

      const layerCanvas = this.layerCanvases[layer]
      if (!layerCanvas) {
        return
      }
      const ctx2 = layerCanvas.getContext('2d')
      ctx2.clearRect(0, 0, layerCanvas.width, layerCanvas.height)
      if (this.layers[layer]) {
        ctx2.putImageData(this.layers[layer], 0, 0)
      }
    },
    newLayer() {
      const data = new ImageData(width, height)
      const i = this.layers.push(data)
      this.currentLayer = i - 1
      return i - 1
    },
    clear(layer) {
      const data = new ImageData(width, height)
      return this.layers[layer] = data
    },
    draw(layer, callback) {
      const canvas = document.createElement("canvas")
      canvas.width = width
      canvas.height = height
      const ctx = canvas.getContext('2d')
      if (this.layers[layer]) {
        ctx.putImageData(this.layers[layer], 0, 0)
      }
      callback(ctx)
      this.layers[layer] = ctx.getImageData(0, 0, width, height)
    },
    rect(layer, x, y, w, h, color) {
      this.draw(layer, ctx => {
        ctx.fillStyle = color
        ctx.fillRect(x, y, w, h)
      })
    },
    outlineRect(layer, x, y, w, h, color, lineWidth=1) {
      this.draw(layer, ctx => {
        ctx.strokeStyle = color
        ctx.lineWidth = lineWidth
        ctx.strokeRect(x, y, w, h)
      })
    },
    point(e) {
      if (e.type === 'mousemove') {
        this.$emit('move', e.offsetX, e.offsetY)
        if (e.which === 1) {
          this.rect(this.currentLayer, e.offsetX, e.offsetY, 1, 1, white)
        }
      } else {
        this.$emit('move', null, null)
      }
    },
  },
  watch: {
    layers: {
      handler(newVal) {
        for (let i = 0; i < newVal.length; i++) {
          this.render(i)
        }
      },
      deep: true,
    },
  },
  beforeUpdate() {
    this.canvases = []
  },
  mounted() {
    const i = this.newLayer()
    this.$nextTick(() => {
      this.rect(i, 10, 10, 20, 20, white)
      this.rect(i, 20, 20, 20, 20, black)
      this.outlineRect(i, 30.5, 30.5, 20, 20, white, 1)
    })
  },
}
</script>

<style scoped>
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

.layers {
  margin: 1rem;
}
.layers ol {
  list-style: none;
  margin: 0;
  padding: 0;
}
.layer-preview {
  width: 60px;
  height: 36px;
  margin-top: 0.25rem;
  margin-bottom: 0.25rem;
  margin-right: 1rem;
  background-color: var(--screen-black);
}
.current {
  color: var(--yellow);
}
</style>
