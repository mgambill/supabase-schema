<template>
  <div
    class="flex items-center border-2 bg-zinc-100 overflow-hidden dark:bg-zinc-800 dark:border-zinc-800 rounded-md opacity-50 hover:opacity-100">
    <button v-tooltip="'Zoom out'" @click="updateView('minus')"
      class="p-3 flex hover:bg-zinc-200 dark:hover:bg-zinc-600 focus:outline-none">
      <i-heroicons-solid:minus-sm></i-heroicons-solid:minus-sm>
    </button>
    <p class="px-3 w-24 border-x-2 border-zinc-800 text-center">
      {{ (state.schemaView.scale * 100).toFixed(0) }}%
    </p>
    <button v-tooltip="'Zoom in'" @click="updateView('plus')"
      class="p-3 flex hover:bg-zinc-200 dark:hover:bg-zinc-600 focus:outline-none">
      <i-heroicons-solid:plus-sm></i-heroicons-solid:plus-sm>
    </button>
  </div>
</template>

<script setup lang="ts">
import { limit, state } from '../store'

const updateView = (type: string) => {
  if (type == 'plus') {
    if (state.schemaView.scale > limit.max) return
    state.schemaView.scale += 0.1
  } else {
    if (state.schemaView.scale <= limit.min) return
    state.schemaView.scale -= 0.1
  }
}
</script>
