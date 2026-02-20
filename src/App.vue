<template>
  <div class="app" :class="`theme-${currentTheme}`">
    <header class="app-header">
      <h1>Kids in Command</h1>
      <p class="subtitle">Choose a theme and explore!</p>
    </header>

    <main class="app-main">
      <WidgetContainer>
        <WidgetCard @openDetails="openDetails('themes', '🎨 Theme Selector')">
          <template #title>🎨 Theme Selector</template>
          <ThemeSelector :themes="themes" v-model="currentTheme" />
        </WidgetCard>

        <WidgetCard @openDetails="openDetails('favorite', '⭐ Favorite Items')">
          <template #title>⭐ Favorite Item</template>
          <FavoriteItemWidget :theme="currentTheme" />
        </WidgetCard>

        <WidgetCard @openDetails="openDetails('weather', '🌤 Weather')">
          <template #title>🌤 Weather</template>
          <WeatherWidget />
        </WidgetCard>
        
        <WidgetCard @openDetails="openDetails('countdown', '📅 Countdown')">
          <template #title>📅 Countdown</template>
          <CountdownWidget />
        </WidgetCard>
        
        <WidgetCard @openDetails="openDetails('mission', '🗺 Mission')">
          <template #title>🗺 Mission</template>
          <MissionWidget />
        </WidgetCard>
        

        <WidgetCard @openDetails="openDetails('rewards', '🏆 Rewards')">
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


    <WidgetDetails v-if="openWidgetId" :title="openWidgetTitle" @close="closeDetails">
      <!-- Show the correct widget in big mode -->
      <CountdownWidget v-if="openWidgetId === 'countdown'" />
      <WeatherWidget v-else-if="openWidgetId === 'weather'" />
      <RewardsWidget v-else-if="openWidgetId === 'rewards'" />
      <FavoriteItemWidget v-else-if="openWidgetId === 'favorite'" :theme="currentTheme" />
      <MissionWidget v-else-if="openWidgetId === 'mission'" />
    </WidgetDetails>
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
import WidgetDetails from "./components/WidgetDetails.vue";



// Theme components (create these later)
import SpaceTheme from "./themes/SpaceTheme.vue";
import DinoTheme from "./themes/DinoTheme.vue";
import PrincessTheme from "./themes/PrincessTheme.vue";
import HeroTheme from "./themes/HeroTheme.vue";

const themes = [
  { id: "space", label: "Space", icon: "🚀" },
  { id: "dino", label: "Dinos", icon: "🦖" },
  { id: "princess", label: "Princess", icon: "👑" },
  { id: "hero", label: "Hero", icon: "🦸" },
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

const openWidgetId = ref("");   // "" means none open
const openWidgetTitle = ref("");


function openDetails(id, title) {
  openWidgetId.value = id;
  openWidgetTitle.value = title;
}
function closeDetails() {
  openWidgetId.value = "";
  openWidgetTitle.value = "";
}
</script>

<style scoped>
.app {
  min-height: 100vh;
  min-width: 100vw;
  font-family: Arial, sans-serif;
  background: #fafafa;
  transition: background 0.5s ease;
  background-size: cover;
  background-position: center;
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


/* Space */
.theme-space {
  background-image: url('/backgrounds/SpaceBackground.png');
}

.theme-space footer {
  color: red;
}

/* Dino */
.theme-dino {
  background-image: url('/backgrounds/DinosaurBackGround.jpg');
}

/* Princess */
.theme-princess {
  background-image: url('/backgrounds/PrincessBackground.webp');
}

/* Hero */
.theme-hero {
  background-image: url('/backgrounds/HeroBackGround.jpg');
}

</style>
