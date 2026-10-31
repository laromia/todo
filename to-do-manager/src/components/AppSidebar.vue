<script setup lang="ts">
defineProps<{
  currentPage: 'overview' | 'tasks' | 'focus';
  totalTasks: number;
}>();

const emit = defineEmits<{
  selectPage: [page: 'overview' | 'tasks' | 'focus'];
}>();
</script>

<template>
  <aside class="sidebar">
    <div class="brand">
      <span class="brand-icon">🌷</span>
      <h2>TaskFlow</h2>
    </div>

    <p class="menu-label">WORKSPACE</p>

    <nav class="navigation">
      <button
        class="nav-item"
        :class="{ active: currentPage === 'overview' }"
        @click="emit('selectPage', 'overview')"
      >
        <span>🏡</span>
        Overview
      </button>

      <button
        class="nav-item"
        :class="{ active: currentPage === 'tasks' }"
        @click="emit('selectPage', 'tasks')"
      >
        <span>🌸</span>
        My Tasks
        <span class="task-count">{{ totalTasks }}</span>
      </button>

      <button
        class="nav-item"
        :class="{ active: currentPage === 'focus' }"
        @click="emit('selectPage', 'focus')"
      >
        <span>⏱️</span>
        Focus
      </button>
    </nav>
  </aside>
</template>