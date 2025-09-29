<script setup lang="ts"></script>

<template>
  <div>
    <div class="hero">
      <div class="hero-content text-center">
        <div class="max-w-md">
          <h1>Hello man!</h1>
          <p>
            这是一个有用的工具，可以帮助你更好地管理你的浏览器。
          </p>

          <div class="flex gap-2 justify-center mb-4">
            <RouterLink
              to="/common/features"
              class="btn btn-primary"
            >
              <i-ph-list-heart />
              Features
            </RouterLink>
            <RouterLink
              to="/common/pricing"
              class="btn btn-primary"
            >
              <i-ph-presentation-chart />
              Pricing
            </RouterLink>
          </div>

          <RouterLink
            to="/common/account/login"
            class="btn btn-secondary btn-lg"
          >
            <i-ph-rocket-launch />
            Get Started Now
          </RouterLink>

          <br />

          <RouterLink
            to="/action-popup/playground"
            class="btn btn-link"
          >
            Playground
          </RouterLink>

          <a
            class="btn btn-link"
            href="https://github.com/mubaidr/vite-vue3-browser-extension-v3"
          >
            Documentation
          </a>

          <a
            class="btn btn-link"
            href="https://mubaidr.js.org"
          >
            Support
          </a>
          <!-- <div class="switch-container">
          <span>开启悬浮对话框</span>
          <label class="switch">
            <input type="checkbox" v-model="isDialogEnabled" @change="toggleDialog">
            <span class="slider round"></span>
          </label>
        </div> -->
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
