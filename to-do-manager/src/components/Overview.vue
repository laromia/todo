<script setup lang="ts">
import { ref, computed, watch } from 'vue';

const props = defineProps<{
  totalTasks: number;
  todoTasks: number;
  inProgressTasks: number;
  completedTasks: number;
}>();

interface CalendarEvent {
  id: number;
  title: string;
  time: string;
  date: string;
  icon: string;
  color: string;
}

const currentDate = ref(new Date());
const selectedDate = ref(new Date());

const monthNames = [
  'January', 'February', 'March', 'April',
  'May', 'June', 'July', 'August',
  'September', 'October', 'November', 'December',
];

const weekDays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];

function dateKey(date: Date): string {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');

  return `${year}-${month}-${day}`;
}

function loadEvents(): CalendarEvent[] {
  try {
    const saved = localStorage.getItem('taskflow-events');
    return saved ? JSON.parse(saved) : [];
  } catch {
    return [];
  }
}

const events = ref<CalendarEvent[]>(loadEvents());

watch(
  events,
  () => {
    localStorage.setItem('taskflow-events', JSON.stringify(events.value));
  },
  { deep: true },
);

// Event form
const showEventForm = ref(false);
const newTitle = ref('');
const newTime = ref('09:00');
const newIcon = ref('🌸');
const newColor = ref('pink');
const eventError = ref('');

const availableIcons = ['🌸', '📚', '☕', '🦋', '💻', '🎀', '✨', '🌷'];
const availableColors = ['pink', 'lavender', 'green', 'peach'];

const selectedEvents = computed(() =>
  events.value
    .filter((event) => event.date === dateKey(selectedDate.value))
    .sort((a, b) => a.time.localeCompare(b.time)),
);

function addEvent() {
  const title = newTitle.value.trim();

  if (!title) {
    eventError.value = 'Please enter an event title.';
    return;
  }

  events.value.push({
    id: Date.now(),
    title,
    time: newTime.value,
    date: dateKey(selectedDate.value),
    icon: newIcon.value,
    color: newColor.value,
  });

  newTitle.value = '';
  newTime.value = '09:00';
  newIcon.value = '🌸';
  newColor.value = 'pink';
  eventError.value = '';
  showEventForm.value = false;
}

function deleteEvent(id: number) {
  events.value = events.value.filter((event) => event.id !== id);
}

const currentMonth = computed(
  () => monthNames[currentDate.value.getMonth()],
);

const currentYear = computed(
  () => currentDate.value.getFullYear(),
);

const calendarDays = computed(() => {
  const year = currentDate.value.getFullYear();
  const month = currentDate.value.getMonth();

  const firstDay = new Date(year, month, 1);
  const startingDay = (firstDay.getDay() + 6) % 7;
  const daysInMonth = new Date(year, month + 1, 0).getDate();

  const days: Date[] = [];

  for (let i = 0; i < startingDay; i++) {
    days.push(new Date(year, month, i - startingDay + 1));
  }

  for (let day = 1; day <= daysInMonth; day++) {
    days.push(new Date(year, month, day));
  }

  while (days.length % 7 !== 0 || days.length < 35) {
    const nextDay = days.length - startingDay - daysInMonth + 1;
    days.push(new Date(year, month + 1, nextDay));
  }

  return days;
});

function previousMonth() {
  currentDate.value = new Date(
    currentYear.value,
    currentDate.value.getMonth() - 1,
    1,
  );
}

function nextMonth() {
  currentDate.value = new Date(
    currentYear.value,
    currentDate.value.getMonth() + 1,
    1,
  );
}

function isToday(date: Date) {
  return date.toDateString() === new Date().toDateString();
}

function isSelected(date: Date) {
  return date.toDateString() === selectedDate.value.toDateString();
}

function isCurrentMonth(date: Date) {
  return date.getMonth() === currentDate.value.getMonth();
}

function hasEvents(date: Date) {
  return events.value.some((event) => event.date === dateKey(date));
}

const formattedSelectedDate = computed(() =>
  selectedDate.value.toLocaleDateString('en-US', {
    weekday: 'long',
    month: 'long',
    day: 'numeric',
  }),
);

const progressPercentage = computed(() =>
  props.totalTasks === 0
    ? 0
    : Math.round((props.completedTasks / props.totalTasks) * 100),
);
</script>

<template>
  <div class="overview">
    <header class="overview-header">
      <div>
        <p class="eyebrow">YOUR PERSONAL SPACE 🌷</p>
        <h1>Welcome back!</h1>
        <p class="subtitle">Let's make today a beautiful day.</p>
      </div>

      <div class="date-badge">
        <span>🗓️</span>
        {{ new Date().toLocaleDateString('en-US', {
          month: 'long',
          day: 'numeric',
          year: 'numeric'
        }) }}
      </div>
    </header>

    <div class="overview-grid">
      <!-- Calendar -->
      <section class="calendar-card">
        <div class="card-heading">
          <div>
            <p class="eyebrow">PLAN YOUR DAYS</p>
            <h2>My calendar ♡</h2>
          </div>

          <div class="calendar-navigation">
            <button type="button" @click="previousMonth" aria-label="Previous month">
              ‹
            </button>
            <button type="button" @click="nextMonth" aria-label="Next month">
              ›
            </button>
          </div>
        </div>

        <div class="month-title">
          {{ currentMonth }} {{ currentYear }}
        </div>

        <div class="calendar-grid">
          <span
            v-for="day in weekDays"
            :key="day"
            class="weekday"
          >
            {{ day }}
          </span>

          <button
  v-for="(date, index) in calendarDays"
  :key="index"
  type="button"
  class="calendar-day"
  :class="{
    'other-month': !isCurrentMonth(date),
    today: isToday(date),
    selected: isSelected(date),
  }"
  @click="selectedDate = date"
>
  {{ date.getDate() }}

  <span v-if="hasEvents(date)" class="event-dot"></span>
  <span v-else-if="isToday(date)" class="today-dot"></span>
</button>
        </div>

        <div class="selected-date">
          <span>🌸</span>
          Selected: {{ formattedSelectedDate }}
        </div>
      </section>

      <!-- Upcoming events -->
      <section class="events-card">
  <div class="card-heading">
    <div>
      <p class="eyebrow">YOUR LITTLE SCHEDULE</p>
      <h2>Daily plans ✨</h2>
      <p class="selected-event-date">{{ formattedSelectedDate }}</p>
    </div>

    <button
      type="button"
      class="add-event-button"
      @click="showEventForm = !showEventForm"
    >
      {{ showEventForm ? 'Cancel' : '+ Add event' }}
    </button>
  </div>

  <!-- Add event form -->
  <form v-if="showEventForm" class="event-form" @submit.prevent="addEvent">
    <label for="event-title">Event title</label>
    <input
      id="event-title"
      v-model="newTitle"
      placeholder="What are you planning?"
    />

    <label for="event-time">Time</label>
    <input id="event-time" v-model="newTime" type="time" />

    <label for="event-icon">Choose an icon</label>
    <select id="event-icon" v-model="newIcon">
      <option v-for="icon in availableIcons" :key="icon" :value="icon">
        {{ icon }}
      </option>
    </select>

    <label for="event-color">Card color</label>
    <select id="event-color" v-model="newColor">
      <option
        v-for="color in availableColors"
        :key="color"
        :value="color"
      >
        {{ color }}
      </option>
    </select>

    <p v-if="eventError" class="event-error">{{ eventError }}</p>

    <button type="submit" class="save-event-button">
      Save event ♡
    </button>
  </form>

  <!-- Events for selected date -->
  <div v-if="selectedEvents.length" class="event-list">
    <div
      v-for="event in selectedEvents"
      :key="event.id"
      class="event-item"
      :class="event.color"
    >
      <span class="event-icon">{{ event.icon }}</span>

      <div class="event-details">
        <p class="event-time">{{ event.time }}</p>
        <h3>{{ event.title }}</h3>
      </div>

      <button
        type="button"
        class="delete-event"
        :aria-label="`Delete ${event.title}`"
        @click="deleteEvent(event.id)"
      >
        ×
      </button>
    </div>
  </div>

  <div v-else class="empty-events">
    <span>☁️</span>
    <p>No plans for this day yet.</p>
    <small>Make something lovely happen!</small>
  </div>
</section>
    </div>

    <!-- Daily focus -->
    <section class="focus-card">
      <div>
        <p class="eyebrow">A LITTLE REMINDER</p>
        <h2>Today's focus 🌼</h2>
        <p>Focus on progress, not perfection. You've got this!</p>
      </div>

      <span class="focus-decoration">🌻</span>
    </section>

    <!-- Productivity -->
    <section class="productivity-card">
      <div class="card-heading">
        <div>
          <p class="eyebrow">LOOK HOW FAR YOU'VE COME</p>
          <h2>Your productivity ♡</h2>
        </div>

        <strong class="percentage">{{ progressPercentage }}%</strong>
      </div>

      <div class="productivity-bar">
        <div :style="{ width: `${progressPercentage}%` }"></div>
      </div>

      <div class="productivity-stats">
        <div>
          <strong>{{ totalTasks }}</strong>
          <span>Total tasks</span>
        </div>

        <div>
          <strong>{{ inProgressTasks }}</strong>
          <span>In progress</span>
        </div>

        <div>
          <strong>{{ completedTasks }}</strong>
          <span>Completed</span>
        </div>

        <div>
          <strong>{{ todoTasks }}</strong>
          <span>To do</span>
        </div>
      </div>
    </section>
  </div>
</template>