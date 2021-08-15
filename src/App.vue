<template>
  <Nav :coords="coords" />
  <Draw @move="move" />
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

    return {
      coords,
      move,
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
  background-color: var(--psd-light-gray);
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
</style>
