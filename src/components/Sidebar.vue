<template>
  <div
    class="fixed top-0 left-0 h-full w-64 bg-gray-800 text-white transform transition-transform duration-300 z-40 rounded-r-lg"
    :class="{ '-translate-x-full': !isSidebarOpen, 'translate-x-0': isSidebarOpen }"
  >
    <!-- Close Button -->
    <div class="flex p-2">
      <div class="flex-none">
        <button @click="closeSidebar" class="btn btn-square btn-ghost">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            class="inline-block h-5 w-5 stroke-current"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M4 6h16M4 12h16M4 18h16"
            ></path>
          </svg>
        </button>
      </div>
      <div class="flex-1">
        <a class="btn btn-ghost text-xl">daisyUI</a>
      </div>
    </div>
    <!-- Sidebar Menu Items -->
    <div class="flex flex-col space-y-2 p-6">
      <RouterLink to="/dashboard" class="border-b border-gray-700 py-2">Dashboard</RouterLink>
      <!-- <RouterLink to="/dashboard" class="border-b border-gray-700 py-2">Bookings</RouterLink>
      <RouterLink to="/service" class="border-b border-gray-700 py-2">Services</RouterLink>
      <RouterLink to="/about" class="border-b border-gray-700 py-2">About</RouterLink> -->
    </div>
  </div>
</template>

<script setup>
import { RouterLink } from 'vue-router'
const props = defineProps({
  isSidebarOpen: {
    type: Boolean,
    required: true,
  },
})

const emit = defineEmits(['closeSidebar'])

const closeSidebar = () => {
  emit('closeSidebar')
}
</script>
