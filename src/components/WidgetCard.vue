<template>
  <article class="card">
    <header class="card-header">
      <h2 class="card-title">
        <slot name="title" />
      </h2>
    </header>

    <div class="card-body">
      <slot />
    </div>
  </article>
</template>

<style scoped>
.card {
  border: 1px solid #e6e6e6;
  border-radius: 14px;
  background: white;
  padding: 14px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.06);
}

.card-header {
  margin-bottom: 10px;
}

.card-title {
  margin: 0;
  font-size: 18px;
}

.card-body {
  font-size: 14px;
}
</style>
