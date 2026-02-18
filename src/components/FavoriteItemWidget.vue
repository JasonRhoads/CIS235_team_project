<template>
  <div class="wrap">
    <p class="label">Favorite Item</p>
    <p class="value">Theme: {{ theme }}</p>
    <p class="hint">Will show theme-based favorites (space/dino/princess/hero).</p>
  </div>
</template>

<script setup>
defineProps({
  theme: { type: String, required: true }
})
</script>

<style scoped>
.label { font-weight: 700; margin: 0 0 6px; }
.value { margin: 0; }
.hint { margin: 8px 0 0; opacity: 0.7; font-size: 13px; }
</style>
