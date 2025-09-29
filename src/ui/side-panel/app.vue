<script setup lang="ts">
import { Notivue, Notification } from 'notivue'
</script>

<template>
  <div class="app-container">
    <AppHeader />

    <div class="p-4 prose main-content">
      <RouterView />
    </div>

    <AppFooter />

    <Notivue v-slot="item">
      <Notification :item="item" />
    </Notivue>
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main-content {
  flex: 1;
}
</style>
