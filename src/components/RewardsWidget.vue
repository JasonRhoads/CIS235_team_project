<template>
  <div class="wrap">
    <div class="top">
      <div class="score">
        <div class="label">Points</div>
        <div class="value">{{ points }}</div>
      </div>

      <button class="earn" @click="earnPoint">
        ⭐ Earn Point
      </button>

      <button class="reset" @click="resetRewards" title="Reset points and badges">
        Reset
      </button>
    </div>

    <!-- little animated +1 bubbles -->
    <div class="pops">
      <TransitionGroup name="pop" tag="div">
        <div v-for="p in pops" :key="p.id" class="pop">+1</div>
      </TransitionGroup>
    </div>

    <div class="badges">
      <div class="badges-title">Badges</div>

      <div v-if="badges.length === 0" class="empty">
        Earn points to unlock badges!
      </div>

      <ul v-else class="list">
        <li v-for="b in badges" :key="b.id" class="badge" :class="{ new: b.isNew }">
          {{ b.icon }} {{ b.name }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const points = ref(0);
const badges = ref([]);
const pops = ref([]);

const badgeRules = [
  { at: 3,  icon: "🥉", name: "Bronze Explorer" },
  { at: 5,  icon: "🥈", name: "Silver Adventurer" },
  { at: 10, icon: "🥇", name: "Gold Champion" },
];

function earnPoint() {
  points.value += 1;

  // Add a quick "+1" popup that disappears
  const id = crypto.randomUUID ? crypto.randomUUID() : String(Date.now() + Math.random());
  pops.value.push({ id });
  setTimeout(() => {
    pops.value = pops.value.filter(x => x.id !== id);
  }, 650);

  // Unlock badges
  const rule = badgeRules.find(r => r.at === points.value);
  if (rule) {
    badges.value.push({
      id: `${rule.at}-${rule.name}`,
      icon: rule.icon,
      name: rule.name,
      isNew: true,
    });

    // remove "new" highlight after a moment
    setTimeout(() => {
      const last = badges.value[badges.value.length - 1];
      if (last) last.isNew = false;
    }, 900);
  }
}

function resetRewards() {
  points.value = 0;
  badges.value = [];
  pops.value = [];
}

// (Optional) persist to localStorage so refreshing keeps progress
watch([points, badges], () => {
  localStorage.setItem("kic_rewards", JSON.stringify({
    points: points.value,
    badges: badges.value.map(b => ({ id: b.id, icon: b.icon, name: b.name })),
  }));
}, { deep: true });

try {
  const saved = JSON.parse(localStorage.getItem("kic_rewards") || "null");
  if (saved) {
    points.value = saved.points ?? 0;
    badges.value = (saved.badges ?? []).map(b => ({ ...b, isNew: false }));
  }
} catch {
  // ignore bad storage
}
</script>

<style scoped>
.wrap {
  position: relative;
}

.top {
  display: grid;
  grid-template-columns: 1fr auto auto;
  gap: 10px;
  align-items: center;
  margin-bottom: 12px;
}

.score .label {
  font-size: 12px;
  opacity: 0.7;
}
.score .value {
  font-size: 28px;
  font-weight: 700;
  line-height: 1;
}

/* Buttons */
.earn, .reset {
  border: none;
  border-radius: 999px;
  padding: 10px 12px;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.earn {
  background: #ff8c00;
  color: white;
  box-shadow: 0 10px 16px rgba(255, 140, 0, 0.25);
}
.earn:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 22px rgba(255, 140, 0, 0.33);
}

.reset {
  background: #f2f2f2;
}
.reset:hover {
  transform: translateY(-1px);
  box-shadow: 0 10px 16px rgba(0, 0, 0, 0.08);
}

/* "+1" popups */
.pops {
  height: 28px;
  overflow: hidden;
  margin-bottom: 10px;
}

.pop {
  display: inline-block;
  margin-right: 8px;
  padding: 4px 8px;
  border-radius: 999px;
  background: rgba(255, 140, 0, 0.15);
  border: 1px solid rgba(255, 140, 0, 0.35);
  font-weight: 700;
}

/* TransitionGroup classes */
.pop-enter-active, .pop-leave-active {
  transition: opacity 0.35s ease, transform 0.35s ease;
}
.pop-enter-from {
  opacity: 0;
  transform: translateY(10px) scale(0.9);
}
.pop-leave-to {
  opacity: 0;
  transform: translateY(-12px) scale(0.9);
}

/* Badges list */
.badges-title {
  font-weight: 700;
  margin-bottom: 6px;
}

.empty {
  opacity: 0.7;
  font-size: 13px;
}

.list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 8px;
}

.badge {
  padding: 10px 12px;
  border-radius: 12px;
  border: 1px solid #e6e6e6;
  background: #fff;
}

/* new badge pulse */
.badge.new {
  border-color: rgba(255, 140, 0, 0.6);
  animation: badgePulse 0.9s ease-in-out 1;
}

@keyframes badgePulse {
  0%   { transform: scale(1); box-shadow: 0 0 0 rgba(255, 140, 0, 0); }
  50%  { transform: scale(1.02); box-shadow: 0 0 18px rgba(255, 140, 0, 0.25); }
  100% { transform: scale(1); box-shadow: 0 0 0 rgba(255, 140, 0, 0); }
}
</style>
