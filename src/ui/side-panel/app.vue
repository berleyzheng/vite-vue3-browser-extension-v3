<script setup lang="ts"></script>

<template>
  <UApp>
    <AppHeader />

    <div class="p-4 prose dark:prose-invert">
      <RouterView />
    </div>

    <AppFooter />
  </UApp>
</template>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: fit-content;
}


.main-content {
  flex: 1;
}
</style>
