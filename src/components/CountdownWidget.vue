<template>
  <div class="wrap">
    <div class="header">
      <div>
        <p class="label">Countdown</p>
        <p class="sub">Next holiday + a few coming up</p>
      </div>
      <button class="refresh" @click="refreshNow">Refresh</button>
    </div>
    <div class="custom">
        <p class="custom-title">Add Your Own Countdown</p>

        <div class="row">
            <input
                class="input"
                v-model.trim="customName"
                placeholder="Event name (e.g., My Birthday)"
            />
            <input
                class="input"
                type="date"
                v-model="customDate"
            />
            <button class="save" @click="saveCustomEvent" :disabled="!canSave">
            Save
            </button>
        </div>

        <div v-if="savedEvent" class="saved">
            <span>Saved:</span>
            <strong>{{ savedEvent.name }}</strong>
            <span>({{ formatShortDate(new Date(savedEvent.date)) }})</span>
            <button class="clear" @click="clearCustomEvent">Clear</button>
        </div>
    </div>

    <p v-if="loading" class="hint">Loading holidays…</p>
    <p v-else-if="error" class="error">{{ error }}</p>

    <div v-else>
      <div v-if="nextHoliday" class="next">
        <p class="next-title">Next Holiday</p>
        <p class="name">{{ nextHoliday.holiday.name }}</p>
        <p class="date">{{ formatDate(nextHoliday.date) }}</p>

        <p class="countdown">
          In <strong>{{ countdownText }}</strong>
        </p>

        <p v-if="nextHoliday.holiday.description" class="desc">
          {{ nextHoliday.holiday.description }}
        </p>
      </div>

      <div v-if="following.length" class="list">
        <p class="next-title">Coming Up</p>
        <ul>
          <li v-for="item in followingLimited" :key="item.holiday.name + item.date.toISOString()">
            <span class="li-name">{{ item.holiday.name }}</span>
            <span class="li-date">{{ formatShortDate(item.date) }}</span>
            <span class="li-days">{{ daysAway(item.date) }}d</span>
          </li>
        </ul>

        <button v-if="following.length > limit" class="more" @click="showAll = !showAll">
          {{ showAll ? "Show fewer" : "Show full list" }}
        </button>
      </div>

      <p v-if="!nextHoliday" class="hint">No upcoming holidays found.</p>
    </div>
  </div>
</template>

<script setup>
    import { computed, onMounted, onUnmounted, ref } from "vue";

    const HOLIDAY_JSON_URL = "/us-holidays.json";

    const loading = ref(true);
    const error = ref("");
    const holidays = ref([]);
    const now = ref(new Date());

    const showAll = ref(false);
    const limit = 3;

    let timerId = null;

    const customName = ref("");
    const customDate = ref(""); // YYYY-MM-DD from <input type="date">
    const savedEvent = ref(null);

    const canSave = computed(() => customName.value && customDate.value);


    // -------------- your helper functions -------- ----------
    function getNthWeekdayOfMonth(year, monthIndex, weekdayIndex, n) 
    {
        if (n > 0) {
            const firstOfMonth = new Date(year, monthIndex, 1);
            const firstDay = firstOfMonth.getDay();
            const offset = (weekdayIndex - firstDay + 7) % 7;
            const day = 1 + offset + 7 * (n - 1);
            return new Date(year, monthIndex, day);
        } else {
            const lastOfMonth = new Date(year, monthIndex + 1, 0);
            const lastDay = lastOfMonth.getDay();
            const offset = (lastDay - weekdayIndex + 7) % 7;
            const day = lastOfMonth.getDate() - offset;
            return new Date(year, monthIndex, day);
        }
    }

    function getEasterSunday(year) {
        const a = year % 19;
        const b = Math.floor(year / 100);
        const c = year % 100;
        const d = Math.floor(b / 4);
        const e = b % 4;
        const f = Math.floor((b + 8) / 25);
        const g = Math.floor((b - f + 1) / 3);
        const h = (19 * a + b - d - g + 15) % 30;
        const i = Math.floor(c / 4);
        const k = c % 4;
        const l = (32 + 2 * e + 2 * i - h - k) % 7;
        const m = Math.floor((a + 11 * h + 22 * l) / 451);
        const month = Math.floor((h + l - 7 * m + 114) / 31);
        const day = ((h + l - 7 * m + 114) % 31) + 1;
        return new Date(year, month - 1, day);
    }

    function getHolidayDateForYear(holiday, year) {
        if (holiday.movable) return getEasterSunday(year);

        const monthIndex = holiday.month - 1;

        if (typeof holiday.day === "number") {
            return new Date(year, monthIndex, holiday.day);
        }

        if (holiday.weekday && typeof holiday.week === "number") {
            const weekdayMap = {
            Sunday: 0, Monday: 1, Tuesday: 2,
            Wednesday: 3, Thursday: 4, Friday: 5, Saturday: 6
            };
            const weekdayIndex = weekdayMap[holiday.weekday];
            return getNthWeekdayOfMonth(year, monthIndex, weekdayIndex, holiday.week);
        }

        return new Date(year, 0, 1);
    }

    function getNextOccurrence(holiday, nowDate) {
        const year = nowDate.getFullYear();
        let dateThisYear = getHolidayDateForYear(holiday, year);

        const todayMid = new Date(nowDate.getFullYear(), nowDate.getMonth(), nowDate.getDate());
        const holidayMid = new Date(dateThisYear.getFullYear(), dateThisYear.getMonth(), dateThisYear.getDate());

        if (holidayMid < todayMid) {
            dateThisYear = getHolidayDateForYear(holiday, year + 1);
        }
        return dateThisYear;
    }

    function formatCountdown(deltaMs) {
        const msPerMinute = 60 * 1000;
        const msPerHour = 60 * msPerMinute;
        const msPerDay = 24 * msPerHour;

        const days = Math.floor(deltaMs / msPerDay);
        const rem1 = deltaMs % msPerDay;
        const hours = Math.floor(rem1 / msPerHour);
        const rem2 = rem1 % msPerHour;
        const mins = Math.floor(rem2 / msPerMinute);

    return { days, hours, mins };
    }

    function saveCustomEvent() {
        if (!canSave.value) return;

        savedEvent.value = {
            name: customName.value,
            date: customDate.value
        };

        localStorage.setItem("kic_custom_event", JSON.stringify(savedEvent.value));
        refreshNow(); // recalc immediately
    }

    function clearCustomEvent() {
        savedEvent.value = null;
        localStorage.removeItem("kic_custom_event");
        refreshNow();
    }


    // ---------- derived data ----------
    const upcoming = computed(() => {
    const n = now.value;
    return holidays.value
        .map(h => {
        const date = getNextOccurrence(h, n);
        const deltaMs = date - n;
        return { holiday: h, date, deltaMs };
        })
        .filter(x => x.deltaMs > 0)
        .sort((a, b) => a.date - b.date);
    });

    const nextHoliday = computed(() => upcoming.value[0] || null);
    const following = computed(() => upcoming.value.slice(1));

    const followingLimited = computed(() =>
    showAll.value ? following.value : following.value.slice(0, limit)
    );

    const countdownText = computed(() => {
        if (!nextHoliday.value) return "";
        const deltaMs = nextHoliday.value.deltaMs;

        const msPerDay = 24 * 60 * 60 * 1000;
        const isClose = deltaMs <= 7 * msPerDay;

        if (!isClose) {
            const daysOnly = Math.ceil(deltaMs / msPerDay);
            return `${daysOnly} day${daysOnly === 1 ? "" : "s"}`;
        }

        const c = formatCountdown(deltaMs);
        return `${c.days} day${c.days === 1 ? "" : "s"}, ${c.hours} hour${c.hours === 1 ? "" : "s"}, ${c.mins} minute${c.mins === 1 ? "" : "s"}`;
    });

    // ---------- formatting helpers ----------
    function formatDate(d) {
        return d.toLocaleDateString(undefined, { weekday: "short", month: "short", day: "numeric", year: "numeric" });
    }
    function formatShortDate(d) {
        return d.toLocaleDateString(undefined, { month: "short", day: "numeric" });
    }
    function daysAway(d) {
        const msPerDay = 24 * 60 * 60 * 1000;
        return Math.ceil((d - now.value) / msPerDay);
    }

    async function refreshNow() {
        now.value = new Date();
        await loadHolidays();
    }   


    // ---------- load data ----------
    async function loadHolidays() {
        loading.value = true;
        error.value = "";

        try {
            const res = await fetch(HOLIDAY_JSON_URL);
            if (!res.ok) throw new Error("Failed to load holidays JSON");
            const data = await res.json();
            holidays.value = (data.holidays || []).slice();
            if (savedEvent.value) {
                const d = new Date(savedEvent.value.date + "T00:00:00");
                holidays.value.push({
                    name: savedEvent.value.name,
                    description: "Your custom countdown event",
                    // Use fixed day/month format:
                    month: d.getMonth() + 1,
                    day: d.getDate()
                });
            }

        } catch (e) {
            error.value = e?.message ?? String(e);
        } finally {
            loading.value = false;
        }
    }

    onMounted(async () => {
        try {
            const saved = JSON.parse(localStorage.getItem("kic_custom_event") || "null");
            if (saved?.name && saved?.date) savedEvent.value = saved;
        } catch {
            // ignore
        }

        await loadHolidays();
        // Update the countdown every minute
        timerId = setInterval(() => {
            now.value = new Date();
        }, 60 * 1000);
    });

    onUnmounted(() => {
        if (timerId) clearInterval(timerId);
    });

</script>

<style scoped>
    /* .wrap { } */
    .header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 10px;
        margin-bottom: 10px;
    }
    .label { 
        font-weight: 800; 
        margin: 0; 
    }
    .sub { 
        margin: 4px 0 0; 
        opacity: 0.7; 
        font-size: 13px; 
    }
    .refresh {
        border: 1px solid #ddd;
        background: #fff;
        border-radius: 999px;
        padding: 8px 10px;
        cursor: pointer;
    }
    .refresh:hover { 
        box-shadow: 0 10px 16px rgba(0,0,0,0.08); 
        transform: translateY(-1px); 
    }

    .next { 
        padding: 10px 0 6px; 
    }
    .next-title { 
        font-weight: 700; 
        margin: 0 0 6px; 
        opacity: 0.8; }
    .name { 
        margin: 0; 
        font-size: 18px; 
        font-weight: 800; 
    }
    .date { 
        margin: 4px 0 0; 
        opacity: 0.75; 
    }
    .countdown { 
        margin: 8px 0 0; 
    }
    .desc { 
        margin: 8px 0 0; 
        opacity: 0.8; 
        font-size: 13px; 
    }

    .list ul { 
        list-style: none; 
        padding: 0; 
        margin: 8px 0 0; 
        display: grid; 
        gap: 8px; 
    }
    .list li {
        display: grid;
        grid-template-columns: 1fr auto auto;
        gap: 10px;
        align-items: center;
        border: 1px solid #eee;
        border-radius: 12px;
        padding: 10px 12px;
    }
    .li-name { 
        font-weight: 700; 
    }
    .li-date { 
        opacity: 0.75; 
        font-size: 13px; 
    }
    .li-days { 
        font-weight: 800; 
    }

    .more {
        margin-top: 10px;
        border: none;
        border-radius: 999px;
        padding: 10px 12px;
        cursor: pointer;
        background: #ff8c00;
        color: white;
    }

    .hint { 
        opacity: 0.7; 
        font-size: 13px; 
        margin: 0; 
    }
    .error { 
        color: #b00020; 
        margin: 0; 
    }

    .custom {
        margin: 12px 0 14px;
        padding: 12px;
        border: 1px dashed #eee;
        border-radius: 12px;
    }

    .custom-title {
        margin: 0 0 10px;
        font-weight: 800;
    }

    .row {
        display: grid;
        grid-template-columns: 1fr 160px auto;
        gap: 10px;
    }

    .input {
        border: 1px solid #ddd;
        border-radius: 10px;
        padding: 10px;
    }

    .save {
        border: none;
        border-radius: 999px;
        padding: 10px 14px;
        cursor: pointer;
        background: #ff8c00;
        color: white;
    }

    .save:disabled {
        opacity: 0.5;
        cursor: not-allowed;
    }

    .saved {
        margin-top: 10px;
        display: flex;
        gap: 8px;
        align-items: center;
        flex-wrap: wrap;
        font-size: 13px;
    }

    .clear {
        border: 1px solid #ddd;
        background: white;
        border-radius: 999px;
        padding: 6px 10px;
        cursor: pointer;
    }

</style>
