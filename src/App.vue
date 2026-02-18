<template>
  <div class="app">
    <header class="app-header">
      <h1>Kids in Command</h1>
      <p class="subtitle">Choose a theme and explore!</p>
    </header>

    <main class="app-main">
      <WidgetContainer>
        <!-- Theme Selector widget -->
        <WidgetCard>
          <template #title>🎨 Theme Selector</template>
          <ThemeSelector :themes="themes" v-model="currentTheme" />
        </WidgetCard>

        <!-- Active Theme widget -->
        <WidgetCard>
          <template #title>🧩 Active Theme</template>

          <!-- dynamic component swap -->
          <component :is="currentThemeComponent" />
        </WidgetCard>

        <!-- Rewards widget (placeholder) -->
        <WidgetCard>
          <template #title>🏆 Rewards</template>
          <p>Rewards coming soon…</p>
        </WidgetCard>
      </WidgetContainer>
    </main>

    <footer class="app-footer">
      <small>Team Project • CIS 235</small>
    </footer>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";
import WidgetContainer from "./components/WidgetContainer.vue";
import WidgetCard from "./components/WidgetCard.vue";
import ThemeSelector from "./components/ThemeSelector.vue";

// Theme components (create these later)
import SpaceTheme from "./themes/SpaceTheme.vue";
import DinoTheme from "./themes/DinoTheme.vue";
import PrincessTheme from "./themes/PrincessTheme.vue";

const themes = [
  { id: "space", label: "Space", icon: "🚀" },
  { id: "dino", label: "Dinos", icon: "🦖" },
  { id: "princess", label: "Princess", icon: "👑" },
];

const currentTheme = ref("space");

const currentThemeComponent = computed(() => {
  switch (currentTheme.value) {
    case "dino":
      return DinoTheme;
    case "princess":
      return PrincessTheme;
    case "space":
    default:
      return SpaceTheme;
  }
});
</script>

<style scoped>
.app {
  min-height: 100vh;
  font-family: Arial, sans-serif;
  background: #fafafa;
  color: #222;
}

.app-header {
  padding: 20px 18px 10px;
  text-align: center;
}

.subtitle {
  margin: 6px 0 0;
  opacity: 0.75;
}

.app-main {
  padding: 14px 18px 22px;
}

.app-footer {
  padding: 14px 18px;
  text-align: center;
  opacity: 0.7;
}
</style>
