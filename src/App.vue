<script setup>
import ScreeningTool from './components/VHDScreeningTool.vue'
</script>

<template>
  <div class="app-container">
    <ScreeningTool />
  </div>
</template>

<style>
.app-container {
  display: flex;
  flex-direction: column;
  padding: 12px;
  max-width: 100%;
}

/* Tablet */
@media (min-width: 768px) {
  .app-container {
    flex-direction: row;
    padding: 20px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .app-container {
    padding: 32px;
  }
}
</style>
