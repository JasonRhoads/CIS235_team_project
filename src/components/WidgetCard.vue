<template>
  <article class="card" @dblclick="$emit('openDetails')">
    <header class="card-header">
      <h2 class="card-title">
        <slot name="title" />
      </h2>

      <button v-if="showDetailsButton" class="details" @click="$emit('openDetails')">
        Details
      </button>
    </header>

    <div class="card-body">
      <slot />
    </div>
  </article>
</template>

<script setup>
defineProps({
  showDetailsButton: { type: Boolean, default: true }
});
defineEmits(["openDetails"]);
</script>

<style scoped>
.card {
  border: 1px solid #e6e6e6;
  border-radius: 14px;
  background: white;
  padding: 14px;
  box-shadow: 0 6px 18px rgba(0,0,0,0.06);
  min-width: 0;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  margin-bottom: 10px;
}

.card-title {
  margin: 0;
  font-size: 18px;
}

.details {
  border: 1px solid #ddd;
  background: #fff;
  border-radius: 999px;
  padding: 8px 10px;
  cursor: pointer;
}

.details:hover {
  box-shadow: 0 10px 16px rgba(0,0,0,0.08);
  transform: translateY(-1px);
}

.card-body {
  font-size: 14px;
}
</style>