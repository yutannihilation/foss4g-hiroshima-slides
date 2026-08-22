<script setup lang="ts">
import { onSlideEnter, onSlideLeave, useSlideContext } from '@slidev/client'
import { ref, watch } from 'vue'

const props = withDefaults(defineProps<{
  emojis?: string[]
  final?: string
  startInterval?: number
  minInterval?: number
  acceleration?: number
}>(), {
  emojis: () => ['🐘', '🦆', '⛰️'],
  final: '🦆',
  startInterval: 500,
  minInterval: 100,
  acceleration: 0.95,
})

const { $clicks } = useSlideContext()
const current = ref(props.final)
let timeoutId: number | undefined
let queue: string[] = []
let isActive = false

function shuffle<T>(items: T[]) {
  const shuffled = [...items]

  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }

  return shuffled
}

function stop() {
  if (timeoutId !== undefined) {
    window.clearTimeout(timeoutId)
    timeoutId = undefined
  }
}

function run() {
  stop()
  queue = []
  let interval = props.startInterval

  function advance() {
    if (queue.length === 0) {
      queue = shuffle(props.emojis)

      if (queue.length > 1 && queue[0] === current.value) {
        ;[queue[0], queue[1]] = [queue[1], queue[0]]
      }
    }

    current.value = queue.shift() ?? props.final
    timeoutId = window.setTimeout(advance, interval)
    interval = Math.max(
      props.minInterval,
      Math.round(interval * props.acceleration),
    )
  }

  advance()
}

function finish() {
  stop()
  current.value = props.final
}

watch($clicks, (clicks) => {
  if (!isActive)
    return

  if (clicks >= 1)
    finish()
  else
    run()
})

onSlideEnter(() => {
  isActive = true

  if ($clicks.value >= 1)
    finish()
  else
    run()
})

onSlideLeave(() => {
  isActive = false
  finish()
})
</script>

<template>
  <div
    class="emoji-roulette"
    role="img"
    aria-label="Emoji roulette ending on DuckDB"
  >
    <span>{{ current }}</span>
  </div>
</template>

<style scoped>
.emoji-roulette {
  display: grid;
  width: 15rem;
  height: 15rem;
  place-items: center;
  font-size: 12rem;
  line-height: 1;
}
</style>
