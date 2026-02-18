<template>
  <div class="row">
    <button
      v-for="t in themes"
      :key="t.id"
      class="btn"
      :class="{ active: modelValue === t.id }"
      @click="$emit('update:modelValue', t.id)"
    >
      <span class="icon">{{ t.icon }}</span>
      {{ t.label }}
    </button>
  </div>
</template>

<script setup>
defineProps({
  themes: { type: Array, required: true },
  modelValue: { type: String, required: true },
});
defineEmits(["update:modelValue"]);
</script>

<style scoped>
.row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.btn {
  border: 1px solid #ddd;
  background: #fff;
  padding: 10px 12px;
  border-radius: 999px;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.08);
}

.btn.active {
  border-color: #ff8c00;
  box-shadow: 0 0 0 3px rgba(255, 140, 0, 0.2);
}

.icon {
  margin-right: 6px;
}
</style>
