<template>
  <div
    :class="vueWorkspaceClasses"
    :style="vueWorkspaceStyle"
  >
    <slot v-bind="{ checkSlotNodes }" />
  </div>
</template>

<script setup>
import { ref, reactive, computed, provide } from 'vue'
import vueWorkspaceTile from './vue-workspace-tile.vue'

const props = defineProps({
  tileSize: {
    type: Number,
    default: 20,
    validator: (value) => value >= 10
  },

  workspaceHeight: {
    type: Number,
    default: 3000
  },

  workspaceWidth: {
    type: Number,
    default: 3000
  }
})

const editable = defineModel('editable', { type: Boolean, default: false })

provide('vueWorkspaceTileSize', props.tileSize)
provide('vueWorkspaceEditable', editable)

const vueWorkspaceClasses = computed(() => [
  'vue-workspace',
  { 'vue-workspace-is-editable': editable.value }
])

const vueWorkspaceStyle = computed(() => [
  { '--vue-workspace-tile-size': `${props.tileSize}px` },
  { '--vue-workspace-height': `${props.workspaceHeight}px` },
  { '--vue-workspace-width': `${props.workspaceWidth}px` }
])

const tiles = reactive([])

function checkSlotNodes(vnode) {
  console.log(vnode)
  return vnode.type === vueWorkspaceTile
}
</script>

<style lang="scss">
.vue-workspace {
  position: relative;
  overflow: hidden;
  display: grid;
  grid-template-rows: repeat(auto-fill, var(--vue-workspace-tile-size));
  grid-template-columns: repeat(auto-fill, var(--vue-workspace-tile-size));

  &.vue-workspace-is-editable {
    width: var(--vue-workspace-width);
    height: var(--vue-workspace-height);
    background-image: radial-gradient(#39303d 10%, transparent 10%);
    background-position: calc(var(--vue-workspace-tile-size) / 2)
      calc(var(--vue-workspace-tile-size) / 2);
    background-size: var(--vue-workspace-tile-size)
      var(--vue-workspace-tile-size);
  }
}
</style>
