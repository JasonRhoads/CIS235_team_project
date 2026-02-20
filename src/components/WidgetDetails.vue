<template>
  <Teleport to="body">
    <div class="backdrop" @click.self="$emit('close')">
      <div class="modal" role="dialog" aria-modal="true">
        <header class="modal-header">
          <h2 class="modal-title">{{ title }}</h2>
          <button class="close" @click="$emit('close')" aria-label="Close">✕</button>
        </header>

        <div class="modal-body">
          <slot />
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
defineProps({
  title: { type: String, default: "Details" }
});
defineEmits(["close"]);
</script>

<style scoped>
.backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.55);
  display: grid;
  place-items: center;
  padding: 18px;
  z-index: 9999;
}

.modal {
  width: min(900px, 96vw);
  max-height: 88vh;
  overflow: auto;
  background: white;
  border-radius: 18px;
  box-shadow: 0 24px 80px rgba(0,0,0,0.35);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 14px 16px;
  border-bottom: 1px solid #eee;
}

.modal-title {
  margin: 0;
  font-size: 18px;
}

.close {
  border: 1px solid #ddd;
  background: #fff;
  border-radius: 10px;
  padding: 6px 10px;
  cursor: pointer;
}

.modal-body {
  padding: 16px;
}
</style>