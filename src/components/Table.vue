<template>
  <div :id="table.title"
    class="selectable pb-2 absolute z-20 box rounded-md overflow-hidden bg-warm-gray-100 dark:bg-dark-700 border-2 dark:border-dark-border !hover:border-green-500"
    :style="{ top: position.y + 'px', left: position.x + 'px' }" style="cursor: grab" @mousedown.prevent="dragStart"
    @mouseenter="isHover = true" @mouseleave="isHover = false">
    <h5
      class="py-2 pb-3 px-2 text-dark-200 dark:text-light-500 bg-warm-gray-200 dark:bg-dark-800 font-medium text-lg text-center border-b-2 dark:border-dark-border">
      <i-bx:bx-news v-if="table.is_view" class="inline mb-1px mr-2"></i-bx:bx-news>
      {{ table?.title }}
    </h5>
    <div v-for="col in table?.columns" :key="col.title">
      <div
        class="py-1 px-4 flex items-center text-dark-100 dark:text-white-800 border-l-3 border-transparent hover:bg-warm-gray-200 dark:hover:bg-dark-600 dark:hover:text-white"
        :class="{ 'border-green-500': col.pk }" :id="`${table?.title}.${col.title}`">
        <p class="flex-grow">
          {{ col.title }}
        </p>
        <p class="ml-10 flex-grow-0 text-sm text-white-900">
          {{ col.format }}
        </p>
      </div>
      <teleport v-if="mounted" to="#canvas-children">
        <Connector v-if="col.fk != undefined" :svg="`svg-${table?.title}.${col.title}`"
          :id="`${table?.title}.${col.title}`" :target="col.fk"></Connector>
      </teleport>
    </div>
  </div>
</template>

<script setup lang="ts">
import { toRefs } from '@vueuse/core'
import {
  computed,
  defineComponent,
  onBeforeMount,
  onMounted,
  ref,
  watch,
} from 'vue'
import type { Table } from '../interface'
import Connector from './Connector.vue'
import { state, supabaseClientState } from '../store'
type Props = {
  table: Table,
  scale: number,
  mounted: Boolean,
}
const props = defineProps<Props>()
const { scale, table } = $(toRefs(props))
const emit = defineEmits(['tableDragging'])

let isDragging = $ref(false)
let ix = $ref(0) //initial
let iy = $ref(0)
let tablesSelected = $ref<any>({})

const position = $computed(() => state.tables[`${table.title}`]?.position ?? 0) // position

// Dragging Event
const dragStart = (e: MouseEvent) => {
  if (e.which !== 1 || e.detail !== 1) return
  emit('tableDragging', true)
  isDragging = true
  document.onmousemove = dragEvent
  document.onmouseup = dragEnd

  if (state.tableSelected.size) {
    state.tableSelected.forEach((el) => {
      tablesSelected[el.id] = {
        ix: e.clientX - state.tables[el.id].position.x * scale,
        iy: e.clientY - state.tables[el.id].position.y * scale,
      }
    })
  } else {
    ix = e.clientX - position.x * scale
    iy = e.clientY - position.y * scale
  }
}
const dragEvent = (e: MouseEvent) => {
  if (!isDragging) return
  if (state.tableSelected.size) {
    state.tableSelected.forEach((el) => {
      state.tables[el.id].position = {
        x: (e.clientX - tablesSelected[el.id].ix) / scale,
        y: (e.clientY - tablesSelected[el.id].iy) / scale,
      }
    })
  } else {
    state.tables[`${table.title}`].position = {
      x: (e.clientX - ix) / scale,
      y: (e.clientY - iy) / scale,
    }
  }
}
const dragEnd = (e: MouseEvent) => {
  emit('tableDragging', false)
  tablesSelected = {}
  isDragging = false
  document.onmousemove = null
  document.onmouseup = null
}

// hover table to highlight connection
const isHover = ref(false)
watch(isHover, (n) => {
  if (n) {
    state.tableHighlighted = table.title
  } else {
    state.tableHighlighted = ''
  }
})
</script>
