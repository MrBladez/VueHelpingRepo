<template>
  <li>
    <span @click="editing = true" v-if="!editing">{{ task.name }}</span>
    <input v-if="editing" v-model="editableName" @blur="updateTask" @keyup.enter="updateTask" />

    <div v-if="!useFlip" :class="['countdown-box', countdownStatus]">
      Due: {{ formattedDate }}<br />
    <strong>{{ countdown }}</strong>
    </div>


    <div v-if="useFlip" :id="'flip-' + task.id"></div>

    <button @click="$emit('remove')">❌</button>
  </li>
</template>


<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import dayjs from 'dayjs'
import duration from 'dayjs/plugin/duration'
import { computed } from 'vue'

dayjs.extend(duration)

const props = defineProps(['task', 'useFlip'])
const emit = defineEmits(['edit', 'remove'])

const editing = ref(false)
const editableName = ref(props.task.name)
const countdown = ref('')
let interval

const countdownStatus = computed(() => {
  const now = dayjs()
  const due = dayjs(props.task.dueDate)
  const diff = due.diff(now, 'hour')

  if (diff < 0) return 'overdue'
  if (diff <= 24) return 'soon'
  return 'safe'
})

const updateCountdown = () => {
  const now = dayjs()
  const due = dayjs(props.task.dueDate)
  const diff = due.diff(now)

  if (diff <= 0) {
    countdown.value = "⏰ Time's up!"
  } else {
    const dur = dayjs.duration(diff)
    const d = Math.floor(dur.asDays())
    const h = dur.hours()
    const m = dur.minutes()
    const s = dur.seconds()
    if (d == 0) {
      if (h == 0) {
        countdown.value = `${m}m ${s}s`
      }else{
        countdown.value = `${h}h ${m}m ${s}s`
      }
    }else if(h == 0) {
      countdown.value = `${d}d ${m}m ${s}s`
    }else{
      countdown.value = `${d}d ${h}h ${m}m ${s}s`
    }
  }
}

onMounted(() => {
  updateCountdown()
  interval = setInterval(updateCountdown, 1000)

  if (props.useFlip) {
    const now = dayjs()
    const due = dayjs(props.task.dueDate)
    const secondsLeft = Math.max(0, due.diff(now, 'second'))

    const clock = new FlipClock(document.getElementById('flip-' + props.task.id), secondsLeft, {
      countdown: true,
      autoStart: true
    })
  }
})

onUnmounted(() => clearInterval(interval))

const formattedDate = dayjs(props.task.dueDate).format("MMMM D, YYYY h:mm A")

const updateTask = () => {
  editing.value = false
  if (editableName.value !== props.task.name) {
    emit('edit', editableName.value)
  }
}

watch(() => props.task.name, newVal => (editableName.value = newVal))

</script>

<style>
  .countdown-box {
  padding: 0.4rem 0.6rem;
  border-radius: 6px;
  font-size: 0.9rem;
  margin-top: 0.3rem;
  }

  .countdown-box.safe {
  background-color: #e0f9e0;
  color: #2a6e2a;
  }

  .countdown-box.soon {
  background-color: #fff4d6;
  color: #996800;
  }

  .countdown-box.overdue {
  background-color: #fce0e0;
  color: #a01919;
  }

</style>