<template>
  <div :id="table.title"
    class="absolute z-20 select-none rounded-lg border-2 border-zinc-200 bg-zinc-100 pb-2 hover:border-emerald-500 dark:border-zinc-700/50 dark:bg-zinc-800 overflow-clip"
    :style="{ top: position.y + 'px', left: position.x + 'px' }" style="cursor: grab" @mousedown.prevent="dragStart"
    @mouseenter="isHover = true" @mouseleave="isHover = false">
    <h5
      class="flex h-12 items-center justify-center rounded-t-lg border-b bg-zinc-200 px-3 text-lg font-medium text-neutral-300 dark:border-zinc-700 dark:bg-zinc-900/10">
      <i-bx:bx-news v-if="table.is_view" class="inline mb-1px mr-2"></i-bx:bx-news>
      {{ table?.title }}
    </h5>
    <template v-for="col in table?.columns" :key="col.title">
      <div
        class="border-l-3 children:py-1.5 children:px-3 flex items-center justify-between gap-x-4 font-medium hover:bg-zinc-700/40"
        :class=" col.pk ? 'border-emerald-500' : 'border-transparent'" :id="`${table?.title}.${col.title}`">
        <dt class="text-zinc-700 dark:text-zinc-400">
          {{ col.title }}
        </dt>
        <dd class="text-sm text-zinc-500">
          {{ col.format }}
        </dd>
      </div>
      <teleport v-if="mounted" to="#canvas-children">
        <Connector v-if="col.fk != undefined" :svg="`svg-${table?.title}.${col.title}`"
          :id="`${table?.title}.${col.title}`" :target="col.fk"></Connector>
      </teleport>
    </template>
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
