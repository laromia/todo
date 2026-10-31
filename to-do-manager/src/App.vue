<script setup lang="ts">
import { ref, computed, watch } from 'vue';

import AppSidebar from './components/AppSidebar.vue';
import AppFooter from './components/AppFooter.vue';
import TaskForm from './components/TaskForm.vue';
import TaskList from './components/TaskList.vue';
import Overview from './components/Overview.vue';
import FocusPage from './components/FocusPage.vue';

import { Status, type Task } from './types/task';

// Load tasks from localStorage
function loadTasks(): Task[] {
  const savedTasks = localStorage.getItem('taskflow-tasks');

  if (!savedTasks) return [];

  try {
    const parsed: unknown = JSON.parse(savedTasks);

    if (!Array.isArray(parsed)) return [];

    return parsed.filter(
      (item): item is Task =>
        typeof item === 'object' &&
        item !== null &&
        typeof item.id === 'number' &&
        typeof item.title === 'string' &&
        Object.values(Status).includes(item.status),
    );
  } catch {
    return [];
  }
}

// Tasks
const tasks = ref<Task[]>(loadTasks());
const searchQuery = ref('');

// Search tasks by title
const searchedTasks = computed(() => {
  return tasks.value.filter((task) =>
    task.title.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// Generate the next task ID
let nextId =
  tasks.value.reduce((maxId, task) => Math.max(maxId, task.id), 0) + 1;

// Navigation
const currentPage = ref<'overview' | 'tasks' | 'focus'>('overview');

// Task counters
const totalTasks = computed(() => tasks.value.length);

const todoTasks = computed(
  () => tasks.value.filter((task) => task.status === Status.TODO).length,
);

const inProgressTasks = computed(
  () => tasks.value.filter((task) => task.status === Status.IN_PROGRESS).length,
);

const completedTasks = computed(
  () => tasks.value.filter((task) => task.status === Status.DONE).length,
);

// Task progress
const progressPercentage = computed(() => {
  if (totalTasks.value === 0) return 0;

  return Math.round(
    (completedTasks.value / totalTasks.value) * 100
  );
});

// Automatically save tasks
watch(
  tasks,
  () => {
    localStorage.setItem('taskflow-tasks', JSON.stringify(tasks.value));
  },
  { deep: true },
);

// Add a new task
function addTask(title: string) {
  tasks.value.push({
    id: nextId++,
    title,
    status: Status.TODO,
  });
}

// Update task status
function updateTaskStatus(id: number, newStatus: Status) {
  const task = tasks.value.find((task) => task.id === id);

  if (task) {
    task.status = newStatus;
  }
}

// Delete a task
function deleteTask(id: number) {
  tasks.value = tasks.value.filter((task) => task.id !== id);
}

// Edit task title
function updateTaskTitle(id: number, newTitle: string) {
  const task = tasks.value.find((task) => task.id === id);
  const trimmedTitle = newTitle.trim();

  if (task && trimmedTitle) {
    task.title = trimmedTitle;
  }
}
</script>

<template>
  <div class="app-layout">

    <!-- SIDEBAR -->
    <AppSidebar
      :current-page="currentPage"
      :total-tasks="totalTasks"
      @select-page="currentPage = $event"
    />

    <!-- MY TASKS PAGE -->
    <main
      v-if="currentPage === 'tasks'"
      class="main-content"
    >
      <section class="task-section">

        <div class="section-heading">
          <div>
            <p class="section-label">LET'S GET THINGS DONE</p>
            <h2>My little tasks ♡</h2>
          </div>

          <span class="flower-decoration">✿</span>
        </div>

        <TaskForm @add-task="addTask" />

        <!-- TASK SEARCH -->
        <div class="task-search">
          <input
            v-model="searchQuery"
            type="search"
            placeholder="Search your tasks..."
            aria-label="Search tasks"
          />
        </div>

        <!-- TASK LIST -->
        <TaskList
          :tasks="searchedTasks"
          @update-status="updateTaskStatus"
          @delete-task="deleteTask"
          @update-title="updateTaskTitle"
        />

      </section>

      <!-- TASK STATISTICS -->
      <section class="stats-grid">

        <article class="stat-card">
          <span>Total tasks</span>
          <strong>{{ totalTasks }}</strong>
        </article>

        <article class="stat-card">
          <span>In progress</span>
          <strong>{{ inProgressTasks }}</strong>
        </article>

        <article class="stat-card">
          <span>Completed</span>
          <strong>{{ completedTasks }}</strong>
        </article>

      </section>

      <!-- DAILY PROGRESS -->
      <section class="progress-card">

        <div class="progress-heading">
          <div>
            <h2>Your daily progress</h2>
            <p>Every little achievement counts ♡</p>
          </div>

          <strong>{{ progressPercentage }}%</strong>
        </div>

        <div
          class="progress-track"
          role="progressbar"
          :aria-valuenow="progressPercentage"
          aria-valuemin="0"
          aria-valuemax="100"
          aria-label="Completed tasks"
        >
          <div
            class="progress-fill"
            :style="{ width: `${progressPercentage}%` }"
          ></div>
        </div>

      </section>

      <AppFooter />
    </main>

    <!-- FOCUS PAGE -->
    <main
      v-else-if="currentPage === 'focus'"
      class="main-content"
    >
      <FocusPage />
    </main>

    <!-- OVERVIEW PAGE -->
    <main
      v-else
      class="main-content"
    >
      <Overview
        :total-tasks="totalTasks"
        :todo-tasks="todoTasks"
        :in-progress-tasks="inProgressTasks"
        :completed-tasks="completedTasks"
      />
    </main>

  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Playfair+Display:wght@500;600;700&display=swap');

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-width: 320px;
  font-family: 'DM Sans', sans-serif;
  background: #fffaf9;
  color: #594653;
}

button,
input,
select {
  font: inherit;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #e9a8bf;
  outline-offset: 3px;
}

.app-layout {
  min-height: 100vh;
  display: flex;
}

.main-content {
  flex: 1;
  min-width: 0;
  max-width: 1250px;
  margin: 0 auto;
  padding: 45px 55px 25px;
}

.welcome-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 35px;
  padding: 30px 35px;
  border-radius: 22px;
  background: linear-gradient(120deg, #fce8ef, #fff0e9, #f4eafa);
}

.welcome-label,
.section-label {
  color: #b77e96;
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 1.5px;
}

.welcome-section h1 {
  margin: 10px 0;
  color: #795568;
  font-family: 'Playfair Display', serif;
  font-size: clamp(1.8rem, 4vw, 2.5rem);
}

.welcome-description {
  margin: 0;
  color: #947d89;
}

.welcome-decoration {
  font-size: 4rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  margin-bottom: 25px;
}

.stat-card {
  position: relative;
  min-height: 155px;
  padding: 22px;
  overflow: hidden;
  border: 1px solid rgb(255 255 255 / 80%);
  border-radius: 18px;
  box-shadow: 0 6px 20px rgb(130 90 110 / 5%);
}

.pink-card {
  background: #fce8ef;
}

.lavender-card {
  background: #eee8fa;
}

.green-card {
  background: #e8f2e8;
}

.stat-icon {
  position: absolute;
  top: 18px;
  right: 20px;
  font-size: 1.5rem;
}

.stat-card p {
  margin: 0 0 8px;
  color: #927487;
  font-size: 0.9rem;
}

.stat-card strong {
  display: block;
  color: #76566b;
  font-family: 'Playfair Display', serif;
  font-size: 2rem;
}

.stat-caption {
  display: block;
  margin-top: 5px;
  color: #a48b9a;
  font-size: 0.75rem;
}

.progress-card {
  margin-bottom: 35px;
  padding: 23px 25px;
  border: 1px solid #f5e5eb;
  border-radius: 18px;
  background: white;
  box-shadow: 0 5px 18px rgb(130 90 110 / 4%);
}

.progress-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.progress-heading h2 {
  margin: 0 0 5px;
  color: #795568;
  font-family: 'Playfair Display', serif;
  font-size: 1.2rem;
}

.progress-heading p {
  margin: 0;
  color: #a48b9a;
  font-size: 0.85rem;
}

.progress-heading strong {
  color: #c16f8d;
  font-size: 1.3rem;
}

.progress-track {
  height: 10px;
  overflow: hidden;
  border-radius: 20px;
  background: #f8eaf0;
}

.progress-fill {
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #e8a3b5, #c89de2);
  transition: width 0.4s ease;
}

.task-section {
  padding: 28px;
  border: 1px solid #f4e5eb;
  border-radius: 22px;
  background: #fff;
  box-shadow: 0 7px 25px rgb(130 90 110 / 4%);
}

.section-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.section-heading h2 {
  margin: 5px 0 0;
  color: #795568;
  font-family: 'Playfair Display', serif;
  font-size: 1.6rem;
}

.flower-decoration {
  color: #e4a1b9;
  font-size: 2rem;
}

.task-form {
  margin: 20px 0;
}

.task-form label {
  display: block;
  margin-bottom: 9px;
  color: #795568;
  font-weight: 600;
}

.form-row {
  display: flex;
  gap: 10px;
}

.form-row input {
  flex: 1;
  min-width: 0;
  padding: 13px 15px;
  border: 1px solid #f0dce5;
  border-radius: 11px;
  background: #fffafc;
}

.form-row input:focus {
  border-color: #d99ab2;
  outline: none;
}

.form-row button {
  padding: 12px 22px;
  border: none;
  border-radius: 11px;
  background: #d987a1;
  color: white;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.2s ease;
}

.form-row button:hover {
  transform: translateY(-2px);
  background: #c87591;
}

.error-message {
  color: #c34d69;
}

.task-list {
  margin-top: 30px;
}

.task-list h2 {
  color: #795568;
  font-family: 'Playfair Display', serif;
}

.task-list ul {
  padding: 0;
  margin: 0;
}

.task-item {
  border: 1px solid #f3e4eb;
  border-radius: 13px;
  background: #fffafd;
  box-shadow: 0 3px 10px rgb(130 90 110 / 3%);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.task-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 15px rgb(130 90 110 / 8%);
}

.task-title {
  color: #684f60;
}

.task-actions select {
  border: 1px solid #edd9e3;
  border-radius: 8px;
  background: white;
  color: #795568;
}

.edit-button {
  background: #eee8fa;
  color: #79619b;
}

.delete-button {
  background: #fce8ef;
  color: #b75d7c;
}

.app-footer {
  margin-top: 30px;
  padding: 15px;
  color: #b18b9b;
  font-size: 0.8rem;
  text-align: center;
}

@media (max-width: 900px) {
  .main-content {
    padding: 30px 25px;
  }

  .stats-grid {
    gap: 12px;
  }
}

@media (max-width: 650px) {
  .app-layout {
    flex-direction: column;
  }

  .main-content {
    padding: 25px 16px;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .welcome-section {
    padding: 22px;
  }

  .welcome-decoration {
    font-size: 2.5rem;
  }

  .task-section {
    padding: 20px 15px;
  }

  .form-row {
    flex-direction: column;
  }
}
</style>