<template>
  <div class="app">
    <header class="app-header">
      <h1>Kids in Command</h1>
      <p class="subtitle">Choose a theme and explore!</p>
    </header>

    <main class="app-main">
      <WidgetContainer>
        <WidgetCard>
          <template #title>🎨 Theme Selector</template>
          <ThemeSelector :themes="themes" v-model="currentTheme" />
        </WidgetCard>

        <WidgetCard>
          <template #title>⭐ Favorite Item</template>
          <FavoriteItemWidget :theme="currentTheme" />
        </WidgetCard>

        <WidgetCard>
          <template #title>🌤 Weather</template>
          <WeatherWidget />
        </WidgetCard>
        
        <WidgetCard>
          <template #title>📅 Countdown</template>
          <CountdownWidget />
        </WidgetCard>
        
        <WidgetCard>
          <template #title>🗺 Mission</template>
          <MissionWidget />
        </WidgetCard>
        

        <WidgetCard>
          <template #title>🏆 Rewards</template>
          <RewardsWidget />
        </WidgetCard>
      </WidgetContainer>

    </main>

    <footer class="app-footer">
      <small>Jason Rhoads and Bobby Don</small>
      <br />
      <small>Team Project • CIS 235</small>
    </footer>
  </div>
</template>

<script setup>
import { computed, ref } from "vue";
import WidgetContainer from "./components/WidgetContainer.vue";
import WidgetCard from "./components/WidgetCard.vue";
import ThemeSelector from "./components/ThemeSelector.vue";
import RewardsWidget from "./components/RewardsWidget.vue";
import CountdownWidget from "./components/CountdownWidget.vue";
import WeatherWidget from "./components/WeatherWidget.vue";
import FavoriteItemWidget from "./components/FavoriteItemWidget.vue";
import MissionWidget from "./components/MissionWidget.vue";



// Theme components (create these later)
import SpaceTheme from "./themes/SpaceTheme.vue";
import DinoTheme from "./themes/DinoTheme.vue";
import PrincessTheme from "./themes/PrincessTheme.vue";
import HeroTheme from "./themes/HeroTheme.vue";

const themes = [
  { id: "space", label: "Space", icon: "🚀" },
  { id: "dino", label: "Dinos", icon: "🦖" },
  { id: "princess", label: "Princess", icon: "👑" },
  { id: "hero", label: "Hero", icon: "👑" },
];

const currentTheme = ref("space");

const currentThemeComponent = computed(() => {
  switch (currentTheme.value) {
    case "dino":
      return DinoTheme;
    case "princess":
      return PrincessTheme;
    case "hero":
      return HeroTheme;
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
