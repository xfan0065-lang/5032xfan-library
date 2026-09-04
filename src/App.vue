<script setup>
import BHeader from './components/BHeader.vue'
</script>

<template>
  <div class="app-shell">
    <BHeader />
    <main>
      <router-view />
    </main>
  </div>
</template>

<style scoped>
.app-shell {
  grid-column: 1 / -1;
  width: 100%;
}
</style>
