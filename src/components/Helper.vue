<template>
  <div class="flex items-center gap-x-3 right-5 bottom-5 absolute z-10 backdrop-blur-sm rounded px-3 border border-zinc-500/50 dark:border-zinc-300/50">
    <button class="text-xl  text-zinc-600 dark:text-zinc-300 btn hover:text-green-600" v-tooltip="'Export Types'" @click="exportTypes = true">
      <i-mdi:language-typescript />
    </button>

    <button class="text-xl  text-zinc-600 dark:text-zinc-300 btn hover:text-green-600" v-tooltip="'Export SQL'" @click="exportSQL = true">
      <i-mdi:database-export-outline />
    </button>

    <button class="text-xl  text-zinc-600 dark:text-zinc-300 btn hover:text-green-600" v-tooltip="'Take a screenshot'" @click="screenshot">
      <i-mdi:camera-outline />
    </button>

    <button class="text-xl  text-zinc-600 dark:text-zinc-300 btn hover:text-green-600" v-tooltip="'Auto arrange'" @click="autoArrange">
      <i-ic:baseline-auto-fix-high />
    </button>

    <button
      class="text-xl  text-zinc-600 dark:text-zinc-300 btn hover:text-green-600"
      v-tooltip="'Focus everything center'"
      @click="focusView"
    >
      <i-uil:focus-target />
    </button>

    <HelperZoom />

    <ModalSQL :open="exportSQL" @close="exportSQL = false" />
    <ModalTypes :open="exportTypes" @close="exportTypes = false" />
  </div>
</template>

<script setup lang="ts">
  import { nextTick, ref } from 'vue'
  import { state } from '../store'
  import { toPng } from 'html-to-image'
  import ModalTypes from './ModalTypes.vue'

  const autoArrange = () => {
    state.autoArrange()
    nextTick(() => {
      focusView()
    })
  }

  const focusView = () => {
    const padding = 100 // padding around the view
    const assumeWidthHeight = 300
    const allX = Object.values(state.tables).map((a) => a.position.x)
    const minX = Math.min.apply(null, allX)
    const maxX = Math.max.apply(null, allX) + assumeWidthHeight
    const allY = Object.values(state.tables).map((a) => a.position.y)
    const minY = Math.min.apply(null, allY)
    const maxY = Math.max.apply(null, allY) + assumeWidthHeight

    const diffX = maxX - minX + padding * 2
    const diffY = maxY - minY + padding * 2
    const scaleX = window.innerWidth / diffX
    const scaleY = window.innerHeight / diffY

    const bestScale = Math.min(scaleX, scaleY)
    state.schemaView.scale = bestScale

    const centeringX = (window.innerWidth - diffX * bestScale) * bestScale
    const centeringY = (window.innerHeight - diffY * bestScale) * bestScale

    const translateX = (-1 * minX + centeringX + padding) * bestScale
    const translateY = (-1 * minY + centeringY + padding) * bestScale

    state.schemaView.translate.x = translateX
    state.schemaView.translate.y = translateY
  }

  const screenshot = () => {
    const el = document.getElementById('screen-canvas') as HTMLElement
    toPng(el).then((dataUrl) => {
      var link = document.createElement('a')
      link.download = 'Supbase Schema.png'
      link.href = dataUrl
      link.click()
    })
  }

  const exportSQL = ref(false)
  const exportTypes = ref(false)
</script>
