<script setup lang="ts">
import { ref } from 'vue';

const title = ref('');
const errorMessage = ref('');

const emit = defineEmits<{
  addTask: [title: string];
}>();

function submitTask() {
  const trimmedTitle = title.value.trim();

  if (!trimmedTitle) {
    errorMessage.value = 'Please enter a task title.';
    return;
  }

  emit('addTask', trimmedTitle);

  title.value = '';
  errorMessage.value = '';
}
</script>

<template>
  <form class="task-form" @submit.prevent="submitTask">
    <label for="task-title">Task title</label>

    <div class="form-row">
      <input
        id="task-title"
        v-model="title"
        type="text"
        placeholder="What do you need to do?"
        :aria-invalid="Boolean(errorMessage)"
        aria-describedby="task-error"
      />

      <button type="submit">Add task</button>
    </div>

    <p
      v-if="errorMessage"
      id="task-error"
      class="error-message"
      role="alert"
    >
      {{ errorMessage }}
    </p>
  </form>
</template>

