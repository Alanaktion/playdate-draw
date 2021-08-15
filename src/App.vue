<template>
  <Nav :coords="coords" :scale="scale" />
  <Draw @move="move" @scale="onScale" />
</template>

<script>
import { ref } from 'vue'
import Nav from './components/Nav.vue'
import Draw from './components/Draw.vue'

export default {
  name: 'App',
  components: {
    Nav,
    Draw,
  },
  setup() {
    const coords = ref([])
    const move = (x, y) => {
      if (x && y) {
        coords.value = [x, y]
      } else {
        coords.value = null
      }
    }
    const scale = ref(1)
    const onScale = (val) => {
      scale.value = val
    }

    return {
      coords,
      move,
      scale,
      onScale,
    }
  }
}
</script>

<style>
:root {
  --screen-black: #312f28;
  --screen-white: #b1afa8;
  --psd-darkest-gray: #212223;
  --psd-mid-gray: #64676c;
  --psd-dark-gray: #7a8085;
  --psd-light-gray: #e8e4da;
  --psd-lightest-gray: #efefef;
  --yellow: #ffc833;
  --device-yellow: #fbc651;
  --purple: #6c00ff;
}
body {
  background-color: var(--psd-lightest-gray);
  color: var(--psd-darkest-gray);
  margin: 0;
  font-family: 'Atkinson Hyperlegible', sans-serif;
}
@media (prefers-color-scheme: dark) {
  body {
    background-color: var(--psd-darkest-gray);
    color: var(--psd-light-gray);
  }
}
@media (min-width: 1200px) {
  html {
    font-size: 20px;
  }
}

.flex {
  display: flex;
}
.items-center {
  align-items: center;
}
.ml-auto {
  margin-left: auto;
}

.transparency-checkerboard {
  background-color: #ff00ff;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='20' height='20' viewBox='0 0 2 2'%3E%3Crect x='0' y='0' width='1' height='1' fill='%23fff'/%3E%3Crect x='1' y='0' width='1' height='1' fill='%23eee'/%3E%3Crect x='0' y='1' width='1' height='1' fill='%23eee'/%3E%3Crect x='1' y='1' width='1' height='1' fill='%23fff'/%3E%3C/svg%3E");
}
</style>
