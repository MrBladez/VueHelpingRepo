<template>
  <transition-group name="fade-slide" tag="ul">
    <TaskItem
      v-for="task in tasks"
      :key="task.id"
      :task="task"
      :useFlip="useFlip"
      @remove="() => $emit('remove-task', task.id)"
      @edit="newName => $emit('edit-task', task.id, newName)"
    />
  </transition-group>
</template>

<script setup>
import TaskItem from './TaskItem.vue'

defineProps(['tasks', 'useFlip'])
defineEmits(['remove-task', 'edit-task'])
</script>

<style scoped>
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.4s ease;
}
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(20px);
}
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}
</style>
