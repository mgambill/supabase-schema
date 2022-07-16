<template>
  <svg
    class="stroke-3 absolute z-10 fill-zinc-400 stroke-zinc-200 dark:fill-zinc-400 dark:stroke-zinc-500 hover:stroke-green-600 dark:hover:stroke-green-600 hover:z-20 group"
    :id="svg" width="0" height="0" pointer-events="none">
    <path :d="attr" pointer-events="visibleStroke" stroke="transparent" fill="transparent" stroke-width="15" />
    <path ref="path" class="fill-transparent" pointer-events="visibleStroke" @mouseenter="isHover = true"
      @mouseleave="isHover = false" />
    <circle :cx="fkPos.x" :cy="fkPos.y" r="7" stroke="none" class="group-hover:fill-green-600"> fk </circle>
  </svg>
</template>

<script setup lang="ts">
import { state } from '../store'
import { computed, onMounted, ref, toRefs, watch } from 'vue'
import { useDark } from '@vueuse/core'

type Props = {
  svg: string
  id: string
  target: string
}

const props = defineProps<Props>()
const { svg, id, target } = $(toRefs(props))

onMounted(() => {
  drawSVG()
})

watch(state.tables, () => drawSVG());

const tableName = $computed(() => id.split('.')[0])
const tableTargetName = $computed(() => target.split('.')[0])
const positionStart = $computed(() => {
  if (state.tables[`${tableName}`]) {
    return state.tables[`${tableName}`].position
  } else {
    return { x: 0, y: 0 }
  }
})
const positionEnd = $computed(() => {
  if (state.tables[`${tableTargetName}`]) {
    return state.tables[`${tableTargetName}`].position
  } else {
    return { x: 0, y: 0 }
  }
})

const fkPos = ref({
  x: 0,
  y: 0,
})

const path = ref<SVGClipPathElement | null>(null)

let attr = $ref("")

const drawSVG = () => {
  if (!path.value) return
  const svgElem = document.getElementById(svg) as HTMLElement
  const startElem = document.getElementById(id) as HTMLElement
  const endElem = document.getElementById(target) as HTMLElement

  const startTop = startElem.offsetTop
  const startHeight = startElem.offsetHeight
  const startLeft = startElem.offsetLeft
  const startRight = startElem.offsetWidth

  const endTop = endElem.offsetTop
  const endHeight = endElem.offsetHeight
  const endLeft = endElem.offsetLeft
  const endRight = endElem.offsetWidth

  const posStartY = positionStart.y + startTop + startHeight / 2
  const posEndY = positionEnd.y + endTop + endHeight / 2
  const posDiffY = posEndY - posStartY
  const posSvgY = posDiffY > 0 ? posStartY : posEndY

  let posStartX = positionStart.x + startLeft
  let posEndX = positionEnd.x + endLeft
  let posDiffX = posEndX - posStartX
  let posSvgX = posDiffX > 0 ? posStartX : posEndX



  if (
    posDiffX + endRight > 0 &&
    posDiffX + endRight < startRight + endRight
  ) {
    // draw
    posStartX = positionStart.x + startRight
    posEndX = positionEnd.x + endRight
    posDiffX = posEndX - posStartX
    posDiffX < 0 ? (posSvgX = posEndX) : (posSvgX = posStartX)

      attr = `M ${Math.abs(posStartX - posSvgX)} ${posDiffY > 0 ? 10 : Math.abs(posDiffY - 10)
      } H ${Math.abs(posDiffX) + 30} V ${posDiffY > 0 ? Math.abs(posDiffY) + 10 : 10
      } H ${Math.abs(posEndX - posSvgX)}`
    path.value.setAttribute('d',attr)

  } else if (posDiffX > 0) {
    posStartX = positionStart.x + startRight
    posEndX = endLeft + positionEnd.x
    posDiffX = posEndX - posStartX
    posSvgX = posStartX

      attr = `M ${Math.abs(posStartX - posSvgX)} ${posDiffY > 0 ? 10 : Math.abs(posDiffY - 10)
      } H ${Math.abs(posStartX - posSvgX + posDiffX / 2)} V ${posDiffY > 0 ? Math.abs(posDiffY) + 10 : 10
      } H ${Math.abs(posEndX - posSvgX)}`
    path.value.setAttribute('d',attr)

  } else {
    posStartX = positionStart.x + startLeft
    posEndX = endRight + positionEnd.x
    posDiffX = posEndX - posStartX
    posSvgX = posEndX
      attr = `M ${Math.abs(posStartX - posSvgX)} ${posDiffY > 0 ? 10 : Math.abs(posDiffY - 10)
      } H ${Math.abs(posStartX - posSvgX + posDiffX / 2)} V ${posDiffY > 0 ? Math.abs(posDiffY) + 10 : 10
      } H ${Math.abs(posEndX - posSvgX)}`

    path.value.setAttribute('d',attr)
  }

  svgElem.style.left = posSvgX + 'px'
  svgElem.style.top = posSvgY - 10 + 'px'
  svgElem.setAttribute('width', `${Math.abs(posDiffX) + 40}`)
  svgElem.setAttribute('height', `${Math.abs(posDiffY) + 20}`)

  fkPos.value.x = posEndX - posSvgX + 2
  fkPos.value.y = posEndY - posSvgY + 10
}

const isHover = ref(false)
watch(
  () => state.tableHighlighted,
  (n) => {
    if (!n) {
      isHover.value = false
    } else if (tableName == n || tableTargetName == n) {
      isHover.value = true
    }
  }
)
</script>
