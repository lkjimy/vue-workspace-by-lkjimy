<template>
  <div class="header">
    <h1 class="title">Vue Workspace</h1>

    <div class="buttons">
      <button @click="editable = !editable">Toggle editing mode</button>

      <button
        @click="addTile"
        :disabled="!editable"
      >
        Add tile
      </button>
    </div>
  </div>

  <div class="workspace-wrapper">
    <vueWorkspace
      :editable="editable"
      :tile-size="20"
    >
      <vueWorkspaceTile
        v-for="tile in tiles"
        :key="tile.name"
        :name="tile.name"
      >
        <div class="my-content">
          {{ tile.name }}
        </div>
      </vueWorkspaceTile>

      <vueWorkspaceTile
        name="custom"
        :x="10"
        :y="10"
        :width="30"
        :height="30"
        :resizeable="false"
      >
        Custom placed tile
      </vueWorkspaceTile>
    </vueWorkspace>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import vueWorkspace from './components/vue-workspace.vue'
import vueWorkspaceTile from './components/vue-workspace-tile.vue'

const editable = ref(false)
const tiles = reactive([])

function addTile() {
  if (!editable.value) return
  const tileCount = tiles.length + 1
  tiles.push({ name: `Tile ${tileCount}`, x: 1, y: 1, width: 5, height: 5 })
}
</script>

<style lang="scss">
:root {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;

  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;

  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  margin: 0;
  min-width: 100vw;
  min-height: 100vh;
}

button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  background-color: #1a1a1a;
  cursor: pointer;
  transition: border-color 0.25s;
}

button:hover {
  border-color: #646cff;
}

button:focus,
button:focus-visible {
  outline: 4px auto -webkit-focus-ring-color;
}

@media (prefers-color-scheme: light) {
  :root {
    color: #213547;
    background-color: #ffffff;
  }
  a:hover {
    color: #747bff;
  }
  button {
    background-color: #f9f9f9;
  }
}

#app {
  --header-height: 60px;

  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 1rem;
    border-bottom: 1px solid #646cff;
    height: var(--header-height);

    .title {
      font-size: 1.2rem;
      font-weight: 600;
      margin: 0;
    }

    .buttons {
      display: flex;
      gap: 1rem;
    }
  }

  .workspace-wrapper {
    height: calc(100vh - var(--header-height));
    width: 100vw;
    overflow: auto;
  }

  .my-content {
    display: flex;
    height: 100%;
    width: 100%;
    justify-content: center;
    align-items: center;
  }
}
</style>
