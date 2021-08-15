<template>
  <div class="playdate">
    <div class="playdate-bezel">
      <div class="canvas-container" @mousemove="point" @mouseleave="point">
        <template v-for="layer in layers" :key="layer.id">
          <canvas
            :ref="el => setCanvasRef(el, layer.id)"
            :width="width"
            :height="height"
            :style="{
              zIndex: layer.index,
            }"
          />
        </template>
      </div>
    </div>
  </div>
  <div class="layers">
    <ol>
      <li
        v-for="layer in sortedLayers"
        :key="layer.id"
        class="flex items-center"
        @click="currentLayer = layer.id"
      >
        <canvas
          :ref="el => setLayerCanvasRef(el, layer.id)"
          :width="width"
          :height="height"
          class="layer-preview"
        />
        <div>
          {{ layer.name }}
          <span class="current" v-if="layer.id === currentLayer">
            (current)
          </span>
        </div>
      </li>
    </ol>
    <button @click="newLayer">New layer</button>
  </div>
</template>

<script>
import { sortBy } from 'lodash'

const width = 400
const height = 240

const white = '#b1afa8'
const black = '#312f28'

let lastId = 0

export default {
  name: 'Draw',
  emits: ['move'],
  data: () => ({
    currentLayer: null,
    layers: {},
    canvases: {},
    layerCanvases: {},
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
    setLayerCanvasRef(el, id) {
      if (el) {
        this.layerCanvases[id] = el
      }
    },
    render(id) {
      // Render full size canvas
      const canvas = this.canvases[id]
      if (!canvas) {
        console.warn(`Can't find canvas for layer ${id}`)
        return
      }

      const { data } = this.layers[id]

      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      if (data) {
        ctx.putImageData(data, 0, 0)
      }

      // Render preview image
      const layerCanvas = this.layerCanvases[id]
      if (!layerCanvas) {
        return
      }
      const ctx2 = layerCanvas.getContext('2d')
      ctx2.clearRect(0, 0, layerCanvas.width, layerCanvas.height)
      if (data) {
        ctx2.putImageData(data, 0, 0)
      }
    },
    newLayer() {
      const data = new ImageData(width, height)
      lastId += 1
      this.layers[lastId] = {
        id: lastId,
        index: Object.keys(this.layers).length,
        name: `Layer ${lastId}`,
        visible: true,
        data,
      }
      this.currentLayer = lastId
      return lastId
    },
    clear(id) {
      const data = new ImageData(width, height)
      return this.layers[id].data = data
    },
    draw(id, callback) {
      const canvas = document.createElement("canvas")
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
    point(e) {
      if (e.type === 'mousemove') {
        this.$emit('move', e.offsetX, e.offsetY)
        if (e.buttons === 1) {
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
        Object.keys(newVal).forEach(id => {
          this.render(id)
        })
      },
      deep: true,
    },
  },
  beforeUpdate() {
    this.canvases = {}
    this.layerCanvases = {}
  },
  mounted() {
    const id = this.newLayer()
    this.$nextTick(() => {
      this.rect(id, 10, 10, 20, 20, white)
      this.rect(id, 20, 20, 20, 20, black)
      this.outlineRect(id, 30.5, 30.5, 20, 20, white, 1)
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
.layers li {
  margin-bottom: 2px;
}
.layer-preview {
  width: 60px;
  height: 36px;
  margin-right: 1rem;
  background-color: var(--screen-black);
}
.layers .current {
  color: var(--yellow);
}
</style>
