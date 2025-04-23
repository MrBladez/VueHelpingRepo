<template>
  <main>
    <h1>📝 My Tasks</h1>
    <TaskForm @add-task="addTask" />
    <select v-model="selectedSubject">
      <option value="">All Subjects</option>
      <option
        v-for="subject in uniqueSubjects"
        :key="subject"
        :value="subject"
      >
      {{ subject }}
      </option>
    </select>

    <TaskList
      :tasks="filteredTasks"
      :useFlip="useFlip"
      @remove-task="removeTask"
      @edit-task="editTask"
    />
    <!-- <button @click="toggleView">
      {{ useFlip ? 'Switch to Text View' : 'Switch to FlipClock' }}
    </button> -->
  </main>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import TaskForm from './components/TaskForm.vue'
import TaskList from './components/TaskList.vue'
import dayjs from 'dayjs'

const tasks = ref([])
const useFlip = ref(false)

const loadTasks = () => {
  const data = localStorage.getItem("tasks")
  if (data) tasks.value = JSON.parse(data)
}

const saveTasks = () => {
  localStorage.setItem("tasks", JSON.stringify(tasks.value))
}

const addTask = (task) => {
  tasks.value.push({ id: Date.now(), ...task })
  saveTasks()
}

const removeTask = (id) => {
  tasks.value = tasks.value.filter(t => t.id !== id)
  saveTasks()
}

const editTask = (id, newName) => {
  const t = tasks.value.find(t => t.id === id)
  if (t) t.name = newName
  saveTasks()
}

const toggleView = () => {
  //useFlip.value = !useFlip.value
}
const selectedSubject = ref('')

const uniqueSubjects = computed(() => {
  const subjects = tasks.value.map(task => task.subject || 'General')
  return [...new Set(subjects)]
})

const filteredTasks = computed(() => {
  if (!selectedSubject.value) return tasks.value
  return tasks.value.filter(task => task.subject === selectedSubject.value)
})


onMounted(() => {
  loadTasks()
  requestNotificationPermission()
  startNotificationCheck()
})

const notifiedTasks = new Set()

const requestNotificationPermission = () => {
  if ('Notification' in window) {
    Notification.requestPermission()
  }
}

const startNotificationCheck = () => {
  setInterval(() => {
    const now = dayjs()
    tasks.value.forEach(task => {
      const due = dayjs(task.dueDate)
      const minutesLeft = due.diff(now, 'minute')

      if (
        minutesLeft <= 10 &&
        minutesLeft >= 0 &&
        !notifiedTasks.has(task.id)
      ) {
        notifiedTasks.add(task.id)
        showNotification(task.name, due)
      }
    })
  }, 1000) // every 1 minute
}

const showNotification = (taskName, due) => {
  if (Notification.permission === 'granted') {
    new Notification('⏰ Task Due Soon', {
      body: `${taskName} is due at ${due.format('h:mm A')}`,
    })
  }
}

</script>
