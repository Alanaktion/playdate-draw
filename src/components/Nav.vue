<template>
  <nav class="flex">
    <div>Playdate Draw</div>
    <div class="flex ml-auto">
      <div v-if="coords" class="highlight" style="margin-right: 1rem;">
        {{ coords[0] }}, {{ coords[1] }}
      </div>
      <div class="highlight">
        {{ scale * 100 }}%
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: 'Nav',
  props: {
    coords: Array,
    scale: Number,
  },
}
</script>

<style scoped>
nav {
  padding: 0.5rem 1rem;
  background-color: var(--psd-light-gray);
}
.highlight {
  color: var(--purple);
}
@media (prefers-color-scheme: dark) {
  nav {
    background-color: var(--screen-black);
  }
  .highlight {
    color: var(--yellow);
  }
}
</style>
