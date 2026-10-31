<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue';

const emit = defineEmits<{
  sessionCompleted: [duration: number];
}>();

// Default durations (in minutes)
const focusMinutes = ref(30);
const breakMinutes = ref(10);

// Timer state
const isFocus = ref(true);
const timeLeft = ref(focusMinutes.value * 60);
const isRunning = ref(false);

let interval: ReturnType<typeof setInterval> | null = null;

// Convert selected durations into seconds
const focusDuration = computed(() => focusMinutes.value * 60);
const breakDuration = computed(() => breakMinutes.value * 60);

const currentDuration = computed(() =>
  isFocus.value ? focusDuration.value : breakDuration.value
);

// Format time as MM:SS
const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60);
  const seconds = timeLeft.value % 60;

  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
});

// Progress bar
const progress = computed(() => {
  return ((currentDuration.value - timeLeft.value) / currentDuration.value) * 100;
});

// Validate and apply the selected durations
function updateDurations() {
  focusMinutes.value = Math.min(180, Math.max(1, Math.floor(Number(focusMinutes.value) || 1)));
  breakMinutes.value = Math.min(180, Math.max(1, Math.floor(Number(breakMinutes.value) || 1)));

  timeLeft.value = currentDuration.value;
}

// Start timer
function startTimer() {
  if (isRunning.value) return;

  isRunning.value = true;

  interval = setInterval(() => {
    if (timeLeft.value > 1) {
      timeLeft.value--;
    } else {
      finishSession();
    }
  }, 1000);
}

// Pause timer
function pauseTimer() {
  if (interval) {
    clearInterval(interval);
  }

  interval = null;
  isRunning.value = false;
}

// Reset current session
function resetTimer() {
  pauseTimer();
  timeLeft.value = currentDuration.value;
}

// Finish current session and switch modes
function finishSession() {
  pauseTimer();

  if (isFocus.value) {
    emit('sessionCompleted', focusDuration.value);
  }

  isFocus.value = !isFocus.value;
  timeLeft.value = currentDuration.value;
}

// Clean up timer when component is destroyed
onUnmounted(() => pauseTimer());
</script>

<template>
  <section class="focus-timer">

    <div class="focus-top">
      <div>
        <span class="focus-eyebrow">YOUR LITTLE PRODUCTIVITY SPACE</span>
        <h2>Focus corner <span>✦</span></h2>
        <p>A little progress is still progress.</p>
      </div>
      <div class="focus-decoration">☁️</div>
    </div>

    <!-- Session mode -->
    <div class="session-switch">
      <div :class="{ selected: isFocus }">
        📚 Focus
      </div>
      <div :class="{ selected: !isFocus }">
        ☕ Break
      </div>
    </div>

    <!-- Duration settings -->
    <div class="timer-settings">
      <label>
        <span>Focus duration</span>
        <div class="duration-input">
          <input
            v-model.number="focusMinutes"
            type="number"
            min="1"
            max="180"
            :disabled="isRunning"
            @change="updateDurations"
          />
          <small>min</small>
        </div>
      </label>

      <label>
        <span>Break duration</span>
        <div class="duration-input">
          <input
            v-model.number="breakMinutes"
            type="number"
            min="1"
            max="180"
            :disabled="isRunning"
            @change="updateDurations"
          />
          <small>min</small>
        </div>
      </label>
    </div>

    <!-- Timer -->
    <div
      class="timer-ring"
      :style="{ '--progress': `${progress}%` }"
    >
      <div class="timer-inner">
        <span class="timer-label">
          {{ isFocus ? 'TIME TO FOCUS' : 'TIME TO REST' }}
        </span>

        <div class="timer-time">{{ formattedTime }}</div>

        <span class="timer-message">
          {{ isFocus ? 'You got this ♡' : 'Take a deep breath ♡' }}
        </span>
      </div>
    </div>

    <p class="timer-next">
      {{ isFocus
        ? `Your next break will be ${breakMinutes} minutes.`
        : `Your next focus session will be ${focusMinutes} minutes.`
      }}
    </p>

    <!-- Controls -->
    <div class="timer-buttons">
      <button
        class="timer-start"
        @click="isRunning ? pauseTimer() : startTimer()"
      >
        {{ isRunning ? 'Ⅱ  Pause' : '▶  Start focusing' }}
      </button>

      <button
        class="timer-reset"
        @click="resetTimer"
        aria-label="Reset timer"
        title="Reset timer"
      >
        ↻
      </button>
    </div>

    <div class="focus-footer">
      <span>✧ One thing at a time ✧</span>
    </div>

  </section>
</template>