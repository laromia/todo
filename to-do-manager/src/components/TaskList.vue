<script setup lang="ts">
import type { Task } from '../types/task';
import { Status } from '../types/task';
import TaskItem from './TaskItem.vue';

defineProps<{
  tasks: Task[];
}>();

const emit = defineEmits<{
  updateStatus: [id: number, status: Status];
  deleteTask: [id: number];
  updateTitle: [id: number, title: string];
}>();
</script>

<template>
  <section class="task-list" aria-labelledby="task-list-title">
    <h2 id="task-list-title">Your tasks</h2>

    <p v-if="tasks.length === 0" class="empty-message">
      No tasks yet. Add your first task above!
    </p>

    <ul v-else>
      <TaskItem
        v-for="task in tasks"
        :key="task.id"
        :task="task"
        @update-status="(id, status) => emit('updateStatus', id, status)"
        @delete-task="(id) => emit('deleteTask', id)"
        @update-title="(id, title) => emit('updateTitle', id, title)"
      />
    </ul>
  </section>
</template>

