<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="taskName" placeholder="Enter a task" required />
    <input v-model="dueDate" type="datetime-local" required />
    <input type="text" v-model="newSubject" placeholder="Subject (e.g. Work)" />
    <button type="submit">Add Task</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'
import dayjs from 'dayjs'

const emit = defineEmits(['add-task'])

const taskName = ref('')
const dueDate = ref('')
const newSubject = ref('')


function handleSubmit() {
  if (taskName.value && dueDate.value) {
    emit('add-task', { name: taskName.value, dueDate: dueDate.value, subject: newSubject.value })
    taskName.value = ''
    dueDate.value = ''
    newSubject.value = ''
  }
}
</script>
