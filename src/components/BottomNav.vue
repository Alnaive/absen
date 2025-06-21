<template>
  <div class="btm-nav md:hidden z-20">
    <RouterLink to="/dashboard" class="border-gray-700 py-2">
      <span class="btm-nav-label">Dashboard</span>
    </RouterLink>
    <RouterLink to="/about" class="border-gray-700 py-2">
      <span class="btm-nav-label">About</span>
    </RouterLink>
  </div>
</template>
<script setup>
import { RouterLink } from 'vue-router'
</script>
