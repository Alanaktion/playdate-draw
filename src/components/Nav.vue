<template>
  <nav class="flex">
    <div>Playdate Draw</div>
    <div class="flex ml-auto">
      <div v-if="coords" class="coords">
        {{ coords[0] }}, {{ coords[1] }}
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: 'Nav',
  props: {
    coords: Array,
  }
}
</script>

<style scoped>
nav {
  padding: 0.5rem 1rem;
  background-color: var(--psd-dark-gray);
}
@media (prefers-color-scheme: dark) {
  nav {
    background-color: var(--screen-black);
  }
}
.coords {
  color: var(--yellow);
}
</style>
