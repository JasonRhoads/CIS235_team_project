<template>
  <section class="grid">
    <slot />
  </section>
</template>

<style scoped>
.grid {
  display: grid;
  gap: 16px;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  align-items: start;
}
</style>
