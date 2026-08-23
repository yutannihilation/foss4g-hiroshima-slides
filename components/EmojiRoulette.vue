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
  minInterval: 70,
  acceleration: 0.95,
})

const { $clicks } = useSlideContext()
const current = ref(props.final)
let timeoutId: number | undefined
let isActive = false

function stop() {
  if (timeoutId !== undefined) {
    window.clearTimeout(timeoutId)
    timeoutId = undefined
  }
}

function run() {
  stop()
  const emojis = props.emojis.length > 0 ? props.emojis : [props.final]
  let index = 0
  let interval = props.startInterval

  function advance() {
    current.value = emojis[index]
    index = (index + 1) % emojis.length
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
