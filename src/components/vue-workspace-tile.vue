<template>
  <div
    class="vue-workspace-tile"
    :class="[
      'vue-workspace-tile',
      { 'is-resizing': isResizing },
      { 'is-resizeable': resizeable && isEditing }
    ]"
    :style="widgetStyle"
  >
    <template v-if="resizeable && isEditing">
      <div class="vue-workspace-tile__info">
        <span class="vue-workspace-tile__info-text">{{ widgetInfo }}</span>
      </div>

      <div class="vue-workspace-tile__handles">
        <div
          class="vue-workspace-tile__handles-top-left"
          @pointerdown="handlePointerDown($event, 'top-left')"
        />

        <div
          class="vue-workspace-tile__handles-top"
          @pointerdown="handlePointerDown($event, 'top')"
        />

        <div
          class="vue-workspace-tile__handles-top-right"
          @pointerdown="handlePointerDown($event, 'top-right')"
        />

        <div
          class="vue-workspace-tile__handles-right"
          @pointerdown="handlePointerDown($event, 'right')"
        />

        <div
          class="vue-workspace-tile__handles-bottom-right"
          @pointerdown="handlePointerDown($event, 'bottom-right')"
        />

        <div
          class="vue-workspace-tile__handles-bottom"
          @pointerdown="handlePointerDown($event, 'bottom')"
        />

        <div
          class="vue-workspace-tile__handles-bottom-left"
          @pointerdown="handlePointerDown($event, 'bottom-left')"
        />

        <div
          class="vue-workspace-tile__handles-left"
          @pointerdown="handlePointerDown($event, 'left')"
        />

        <div
          class="vue-workspace-tile__handles-center"
          @pointerdown="handlePointerDown($event, 'center')"
        />
      </div>
    </template>

    <div class="vue-workspace-tile__main">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, defineProps, inject } from 'vue'

const props = defineProps({
  name: {
    type: String,
    required: true
  },

  x: {
    type: Number,
    default: 1,
    validator: (value) => value >= 1
  },

  y: {
    type: Number,
    default: 1,
    validator: (value) => value >= 1
  },

  width: {
    type: Number,
    default: 5,
    validator: (value) => value >= 5
  },

  height: {
    type: Number,
    default: 5,
    validator: (value) => value >= 5
  },

  resizeable: {
    type: Boolean,
    default: true
  }
})

const isResizing = ref(false)
const isEditing = inject('vueWorkspaceEditable', false) /* */ // Is widget in edit mode
const TILE_SIZE = inject('vueWorkspaceTileSize', 20) /*    */ // Grid size in px. Value from workspace.
const MIN_WIDTH = 5 /*                                     */ // Minimum width in grid blocks
const MIN_HEIGHT = 5 /*                                    */ // Minimum height in grid blocks
const MIN_X = 1 /*                                         */ // Minimum X position in layout grid
const MIX_Y = 1 /*                                         */ // Minimum Y position in layout grid

const widgetStyle = ref({
  '--pos-y': props.y,
  '--pos-x': props.x,
  '--width': props.width,
  '--height': props.height
})

const widgetInfo = computed(() => {
  return `(${widgetStyle.value['--pos-x']} ${widgetStyle.value['--pos-y']}) ${widgetStyle.value['--width']}x${widgetStyle.value['--height']}`
})

function setWidgetSize(widgetSizeObj) {
  if (!widgetSizeObj) return

  if (widgetSizeObj.width >= MIN_WIDTH) {
    widgetStyle.value['--width'] = widgetSizeObj.width
  }

  if (widgetSizeObj.height >= MIN_HEIGHT) {
    widgetStyle.value['--height'] = widgetSizeObj.height
  }

  if (widgetSizeObj.y >= MIX_Y) {
    widgetStyle.value['--pos-y'] = widgetSizeObj.y
  }

  if (widgetSizeObj.x >= MIN_X) {
    widgetStyle.value['--pos-x'] = widgetSizeObj.x
  }
}

const initialPosX = ref(null)
const initialPosY = ref(null)

function handlePointerDown(mdEvent, handle) {
  if (!props.resizeable) return

  isResizing.value = true
  console.log('PointerDown:', mdEvent)

  // Save initial position
  initialPosX.value = mdEvent.clientX
  initialPosY.value = mdEvent.clientY

  function handlePointerMove(event) {
    resize(event, handle)
  }

  window.addEventListener('pointermove', handlePointerMove)
  window.addEventListener(
    'pointerup',
    () => {
      window.removeEventListener('pointermove', handlePointerMove)
      isResizing.value = false
      console.log('stopResize')
    },
    { once: true }
  )
}

function resize(event, handle) {
  // Calculate how much the mouse has moved in px
  const movedX = event.clientX - initialPosX.value
  const movedY = event.clientY - initialPosY.value

  // Calculate how many blocks the widget has to be changed
  let movedX_inBlocks = 0
  let movedY_inBlocks = 0

  if (movedX >= TILE_SIZE) {
    // movedX_inBlocks = Math.floor(movedX / BLOCK_SIZE)
    movedX_inBlocks = Math.round(movedX / TILE_SIZE)
    // movedX_inBlocks++
  }

  if (movedX <= TILE_SIZE * -1) {
    // movedX_inBlocks = Math.ceil(movedX / BLOCK_SIZE)
    movedX_inBlocks = Math.round(movedX / TILE_SIZE)
    // movedX_inBlocks--
  }

  if (movedY >= TILE_SIZE) {
    // movedY_inBlocks = Math.floor(movedY / BLOCK_SIZE)
    movedY_inBlocks = Math.round(movedY / TILE_SIZE)
    // movedY_inBlocks++
  }

  if (movedY <= TILE_SIZE * -1) {
    // movedY_inBlocks = Math.ceil(movedY / BLOCK_SIZE)
    movedY_inBlocks = Math.round(movedY / TILE_SIZE)
    // movedY_inBlocks--
  }

  // Check if the widget has to be resized
  if (movedX_inBlocks === 0 && movedY_inBlocks === 0) return

  console.log(
    'Resize: ',
    handle,
    // event,
    { initialPosX: initialPosX.value, initialPosY: initialPosY.value },
    { movedX, movedY },
    { movedX_inBlocks, movedY_inBlocks }
  )

  // Check which handle is being moved
  switch (handle) {
    case 'top-left':
      break
    case 'top':
      setWidgetSize({
        y: widgetStyle.value['--pos-y'] + movedY_inBlocks,
        height: widgetStyle.value['--height'] + movedY_inBlocks
      })

      break
    case 'top-right':
      break
    case 'right':
      setWidgetSize({
        width: widgetStyle.value['--width'] + movedX_inBlocks
      })
      break
    case 'bottom-right':
      setWidgetSize({
        width: widgetStyle.value['--width'] + movedX_inBlocks,
        height: widgetStyle.value['--height'] + movedY_inBlocks
      })
      break
    case 'bottom':
      setWidgetSize({
        height: widgetStyle.value['--height'] + movedY_inBlocks
      })
      break
    case 'bottom-left':
      break
    case 'left':
      setWidgetSize({
        x: widgetStyle.value['--pos-x'] - movedX_inBlocks,
        width: widgetStyle.value['--width'] + movedX_inBlocks
      })
      break
    case 'center':
      setWidgetSize({
        x: widgetStyle.value['--pos-x'] + movedX_inBlocks,
        y: widgetStyle.value['--pos-y'] + movedY_inBlocks
      })
      break
  }

  initialPosX.value = event.clientX
  initialPosY.value = event.clientY
}
</script>

<style lang="scss">
.vue-workspace-tile {
  position: relative;
  background-color: cadetblue;
  // border: 1px solid cadetblue;
  border-radius: 4px;
  // padding: 10px;
  grid-row: var(--pos-y, 1) / span var(--height, 3);
  grid-column: var(--pos-x, 1) / span var(--width, 3);
  overflow: hidden;
  z-index: 1;

  &__info,
  &__handles {
    display: none;
  }

  &.is-resizing {
    z-index: 2;
  }

  &.is-resizeable {
    &.is-resizing .vue-workspace-tile__info {
      position: absolute;
      display: block;
      bottom: 10px;
      right: 10px;
      z-index: 1;
      pointer-events: none;
      user-select: none;
      background: rgba(255, 255, 255, 0.5);
      backdrop-filter: blur(10px);
      padding: 2px 4px;
      border-radius: 4px;
    }

    .vue-workspace-tile__handles {
      display: block;
      opacity: 0.3;

      & > * {
        position: absolute;
        z-index: 3;
      }

      &-top-left {
        top: 0;
        left: 0;
        width: 10px;
        height: 10px;
        background-color: red;
        cursor: nwse-resize;
      }

      &-top {
        top: 0;
        left: 10px;
        right: 10px;
        height: 10px;
        background-color: blue;
        cursor: ns-resize;
      }

      &-top-right {
        top: 0;
        right: 0;
        width: 10px;
        height: 10px;
        background-color: red;
        cursor: nesw-resize;
      }

      &-right {
        top: 10px;
        bottom: 10px;
        right: 0;
        width: 10px;
        background-color: blue;
        cursor: ew-resize;
      }

      &-bottom-right {
        bottom: 0;
        right: 0;
        width: 10px;
        height: 10px;
        background-color: red;
        cursor: nwse-resize;
      }

      &-bottom {
        bottom: 0;
        left: 10px;
        right: 10px;
        height: 10px;
        background-color: blue;
        cursor: ns-resize;
      }

      &-bottom-left {
        bottom: 0;
        left: 0;
        width: 10px;
        height: 10px;
        background-color: red;
        cursor: nesw-resize;
      }

      &-left {
        top: 10px;
        bottom: 10px;
        left: 0;
        width: 10px;
        background-color: blue;
        cursor: ew-resize;
      }

      &-center {
        top: 10px;
        left: 10px;
        right: 10px;
        bottom: 10px;
        background-color: teal;
        cursor: all-scroll;
      }
    }

    .res-widget__main {
      pointer-events: none;
      user-select: none;
    }
  }

  &__main {
    display: block;
    height: 100%;
    width: 100%;
    z-index: 1;
  }
}
</style>
