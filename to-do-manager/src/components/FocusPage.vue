<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import FocusTimer from './FocusTimer.vue';

interface FocusSession {
  id: number;
  date: string;
  duration: number; // Duration in seconds
}

const STORAGE_KEY = 'taskflow-focus-sessions';

function loadSessions(): FocusSession[] {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) return [];

    const parsed: unknown = JSON.parse(saved);

    if (!Array.isArray(parsed)) return [];

    return parsed.filter(
      (session): session is FocusSession =>
        typeof session.id === 'number' &&
        typeof session.date === 'string' &&
        typeof session.duration === 'number'
    );
  } catch {
    return [];
  }
}

const sessions = ref<FocusSession[]>(loadSessions());

watch(
  sessions,
  () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(sessions.value));
  },
  { deep: true }
);

function addFocusSession(duration: number) {
  sessions.value.unshift({
    id: Date.now(),
    date: new Date().toISOString(),
    duration
  });
}

function isToday(date: string): boolean {
  return new Date(date).toDateString() === new Date().toDateString();
}

const todaySessions = computed(() =>
  sessions.value.filter(session => isToday(session.date))
);

const todayFocusTime = computed(() =>
  todaySessions.value.reduce((total, session) => total + session.duration, 0)
);

const todayHours = computed(() =>
  Math.floor(todayFocusTime.value / 3600)
);

const todayMinutes = computed(() =>
  Math.floor((todayFocusTime.value % 3600) / 60)
);

const weeklyStats = computed(() => {
  const today = new Date();
  const days = [];

  for (let i = 6; i >= 0; i--) {
    const date = new Date(today);
    date.setHours(0, 0, 0, 0);
    date.setDate(today.getDate() - i);

    const dailySessions = sessions.value.filter(session => {
      const sessionDate = new Date(session.date);
      return sessionDate.toDateString() === date.toDateString();
    });

    const total = dailySessions.reduce(
      (sum, session) => sum + session.duration,
      0
    );

    days.push({
      label: date.toLocaleDateString('en', { weekday: 'short' }),
      minutes: Math.floor(total / 60)
    });
  }

  return days;
});

const weeklyTotal = computed(() =>
  weeklyStats.value.reduce((total, day) => total + day.minutes, 0)
);

const maxWeeklyMinutes = computed(() =>
  Math.max(...weeklyStats.value.map(day => day.minutes), 60)
);

function formatDuration(seconds: number): string {
  const minutes = Math.floor(seconds / 60);
  const hours = Math.floor(minutes / 60);
  const remainingMinutes = minutes % 60;

  if (hours > 0) return `${hours}h ${remainingMinutes}m`;
  return `${minutes} min`;
}

function formatSessionDate(date: string): string {
  return new Date(date).toLocaleString('en', {
    day: 'numeric',
    month: 'short',
    hour: '2-digit',
    minute: '2-digit'
  });
}
</script>

<template>
  <div class="focus-page">
    <header class="focus-page-header">
      <div>
        <p class="section-label">A LITTLE TIME FOR YOURSELF ✦</p>
        <h1>Focus space ♡</h1>
        <p class="focus-subtitle">
          Slow down, concentrate, and make progress one session at a time.
        </p>
      </div>
      <span class="focus-decoration">☁️</span>
    </header>

    <section class="focus-main-card">
      <div class="focus-card-heading">
        <span>✦ YOUR FOCUS SESSION ✦</span>
      </div>

      <FocusTimer @session-completed="addFocusSession" />
    </section>

    <section class="focus-summary">
      <article class="focus-stat-card">
        <span class="stat-icon">⏳</span>
        <p>Today's focus time</p>
        <h2>{{ todayHours }}h {{ todayMinutes }}m</h2>
      </article>

      <article class="focus-stat-card">
        <span class="stat-icon">🌸</span>
        <p>Completed sessions</p>
        <h2>{{ todaySessions.length }}</h2>
      </article>

      <article class="focus-stat-card">
        <span class="stat-icon">✨</span>
        <p>This week's focus</p>
        <h2>{{ formatDuration(weeklyTotal * 60) }}</h2>
      </article>
    </section>

    <section class="weekly-card">
      <div class="focus-section-heading">
        <div>
          <p class="section-label">YOUR PROGRESS</p>
          <h2>Weekly activity</h2>
        </div>
        <span>☁️</span>
      </div>

      <div class="weekly-chart">
        <div
          v-for="day in weeklyStats"
          :key="day.label"
          class="chart-day"
        >
          <div class="chart-bar-container">
            <div
              class="chart-bar"
              :style="{
                height: `${(day.minutes / maxWeeklyMinutes) * 100}%`
              }"
            ></div>
          </div>

          <span class="chart-minutes">{{ day.minutes }}m</span>
          <span class="chart-label">{{ day.label }}</span>
        </div>
      </div>
    </section>

    <section class="history-card">
      <div class="focus-section-heading">
        <div>
          <p class="section-label">YOUR LITTLE ACHIEVEMENTS</p>
          <h2>Focus history</h2>
        </div>
        <span>♡</span>
      </div>

      <div v-if="sessions.length === 0" class="empty-history">
        <span>🌷</span>
        <p>No sessions yet.</p>
        <small>Your completed focus sessions will appear here.</small>
      </div>

      <div v-else class="history-list">
        <article
          v-for="session in sessions"
          :key="session.id"
          class="history-item"
        >
          <span class="history-icon">📚</span>

          <div class="history-details">
            <strong>Focus session</strong>
            <small>{{ formatSessionDate(session.date) }}</small>
          </div>

          <span class="history-duration">
            {{ formatDuration(session.duration) }}
          </span>
        </article>
      </div>
    </section>
  </div>
</template>