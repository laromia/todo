<script setup lang="ts">
import { ref } from 'vue';
import { Status, type Task } from '../types/task';

const props = defineProps<{
  task: Task;
}>();

const isEditing = ref(false);
const editedTitle = ref(props.task.title);
const editError = ref('');

const emit = defineEmits<{
  updateStatus: [id: number, status: Status];
  deleteTask: [id: number];
  updateTitle: [id: number, title: string];
}>();

function changeStatus(event: Event) {
  const select = event.target as HTMLSelectElement;

  emit('updateStatus', props.task.id, select.value as Status);
}

function deleteCurrentTask() {
  emit('deleteTask', props.task.id);
}

function startEditing() {
  editedTitle.value = props.task.title;
  editError.value = '';
  isEditing.value = true;
}

function saveTitle() {
  const trimmedTitle = editedTitle.value.trim();

  if (!trimmedTitle) {
    editError.value = 'The task title cannot be empty.';
    return;
  }

  emit('updateTitle', props.task.id, trimmedTitle);

  isEditing.value = false;
  editError.value = '';
}

function cancelEditing() {
  editedTitle.value = props.task.title;
  editError.value = '';
  isEditing.value = false;
}
</script>

<template>
  <li class="task-item">
    <div class="task-content">
      <div v-if="isEditing" class="edit-section">
        <label :for="`edit-task-${task.id}`">Edit task title</label>

        <input
          :id="`edit-task-${task.id}`"
          v-model="editedTitle"
          type="text"
          @keyup.enter="saveTitle"
          @keyup.esc="cancelEditing"
        />

        <div class="edit-actions">
          <button type="button" @click="saveTitle">Save</button>
          <button type="button" @click="cancelEditing">Cancel</button>
        </div>

        <p v-if="editError" class="error-message" role="alert">
          {{ editError }}
        </p>
      </div>

      <div v-else class="task-title-section">
        <span
          class="task-title"
          :class="{ completed: task.status === Status.DONE }"
        >
          {{ task.title }}
        </span>

        <button
          type="button"
          class="edit-button"
          :aria-label="`Edit ${task.title}`"
          @click="startEditing"
        >
          Edit
        </button>
      </div>

      <div class="task-actions">
        <label :for="`status-${task.id}`" class="visually-hidden">
          Status for {{ task.title }}
        </label>

        <select
          :id="`status-${task.id}`"
          :value="task.status"
          @change="changeStatus"
        >
          <option :value="Status.TODO">To do</option>
          <option :value="Status.IN_PROGRESS">In progress</option>
          <option :value="Status.DONE">Done</option>
        </select>

        <button
          type="button"
          class="delete-button"
          :aria-label="`Delete ${task.title}`"
          @click="deleteCurrentTask"
        >
          Delete
        </button>
      </div>
    </div>
  </li>
</template>

