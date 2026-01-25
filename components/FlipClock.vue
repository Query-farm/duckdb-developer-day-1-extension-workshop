<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue'

const props = defineProps({
  minutes: {
    type: Number,
    default: 10,
  },
  seconds: {
    type: Number,
    default: 0,
  },
  autoStart: {
    type: Boolean,
    default: false,
  },
})

const initialTime = props.minutes * 60 + props.seconds
const timeRemaining = ref(initialTime)
const isRunning = ref(false)
const isFinished = ref(false)
let intervalId: number | null = null

// Track which digits are animating
const animating = ref([false, false, false, false])

const displayMinutes = computed(() => String(Math.floor(timeRemaining.value / 60)).padStart(2, '0'))
const displaySeconds = computed(() => String(timeRemaining.value % 60).padStart(2, '0'))

const allDigits = computed(() => [
  displayMinutes.value[0],
  displayMinutes.value[1],
  displaySeconds.value[0],
  displaySeconds.value[1],
])

let prevDigits = [...allDigits.value]

function checkForChanges() {
  const current = allDigits.value
  current.forEach((digit, i) => {
    if (digit !== prevDigits[i]) {
      animating.value[i] = true
      setTimeout(() => {
        animating.value[i] = false
      }, 150)
    }
  })
  prevDigits = [...current]
}

function startTimer() {
  if (intervalId !== null || timeRemaining.value <= 0) return
  isRunning.value = true
  isFinished.value = false
  intervalId = window.setInterval(() => {
    if (timeRemaining.value > 0) {
      timeRemaining.value--
      checkForChanges()
    } else {
      stopTimer()
      isFinished.value = true
    }
  }, 1000)
}

function stopTimer() {
  if (intervalId !== null) {
    clearInterval(intervalId)
    intervalId = null
  }
  isRunning.value = false
}

function resetTimer() {
  stopTimer()
  timeRemaining.value = initialTime
  isFinished.value = false
  prevDigits = allDigits.value.slice()
}

function toggleTimer() {
  isRunning.value ? stopTimer() : startTimer()
}

function addMinute() {
  timeRemaining.value += 60
  checkForChanges()
}

function subtractMinute() {
  if (timeRemaining.value >= 60) {
    timeRemaining.value -= 60
    checkForChanges()
  }
}

onUnmounted(() => {
  if (intervalId !== null) clearInterval(intervalId)
})

if (props.autoStart) startTimer()
</script>

<template>
  <div class="clock-container">
    <div class="clock" :class="{ finished: isFinished }">
      <div class="digit-group">
        <div class="digit" :class="{ changing: animating[0] }">{{ allDigits[0] }}</div>
        <div class="digit" :class="{ changing: animating[1] }">{{ allDigits[1] }}</div>
      </div>

      <div class="separator">:</div>

      <div class="digit-group">
        <div class="digit" :class="{ changing: animating[2] }">{{ allDigits[2] }}</div>
        <div class="digit" :class="{ changing: animating[3] }">{{ allDigits[3] }}</div>
      </div>
    </div>

    <div class="controls">
      <button class="btn time-adjust" @click="subtractMinute" title="Subtract 1 minute">−1m</button>
      <button class="btn" :class="{ active: isRunning }" @click="toggleTimer">
        {{ isRunning ? '⏸ Pause' : '▶ Start' }}
      </button>
      <button class="btn reset" @click="resetTimer">↺ Reset</button>
      <button class="btn time-adjust" @click="addMinute" title="Add 1 minute">+1m</button>
    </div>
  </div>
</template>

<style scoped>
.clock-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
}

.clock {
  display: flex;
  align-items: center;
  gap: 8px;
}

.clock.finished {
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.digit-group {
  display: flex;
  gap: 6px;
}

.digit {
  width: 72px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(180deg, #333 0%, #1a1a1a 100%);
  border-radius: 8px;
  font-family: 'SF Mono', Monaco, monospace;
  font-size: 64px;
  font-weight: 700;
  color: #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  transition: opacity 0.15s ease-out, filter 0.1s ease-out;
}

.digit.changing {
  opacity: 0.6;
  filter: blur(1px);
}

.separator {
  font-family: 'SF Mono', Monaco, monospace;
  font-size: 64px;
  font-weight: 700;
  color: #fff;
  animation: blink 1s infinite;
  margin: 0 4px;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.3; }
}

.controls {
  display: flex;
  gap: 12px;
}

.btn {
  padding: 12px 24px;
  font-size: 16px;
  font-weight: 600;
  border: 1px solid rgba(255,255,255,0.2);
  border-radius: 8px;
  background: rgba(255,255,255,0.15);
  color: #fff;
  cursor: pointer;
  backdrop-filter: blur(10px);
  transition: all 0.2s;
}

.btn:hover {
  background: rgba(255,255,255,0.25);
  transform: translateY(-2px);
}

.btn.active {
  background: rgba(234,179,8,0.3);
  border-color: rgba(234,179,8,0.5);
}

.btn.reset {
  background: rgba(239,68,68,0.2);
  border-color: rgba(239,68,68,0.3);
}

.btn.reset:hover {
  background: rgba(239,68,68,0.3);
}

.btn.time-adjust {
  background: rgba(59,130,246,0.2);
  border-color: rgba(59,130,246,0.3);
  font-weight: 700;
  min-width: 50px;
}

.btn.time-adjust:hover {
  background: rgba(59,130,246,0.3);
}
</style>
