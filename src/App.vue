<template>
  <div class="relative">
    <!-- Navbar with toggle button -->
    <Navbar :isSidebarOpen="isSidebarOpen" @toggleSidebar="toggleSidebar" />

    <!-- Sidebar -->
    <Sidebar :isSidebarOpen="isSidebarOpen" @closeSidebar="closeSidebar" />

    <!-- Bottom Nav-->
    <!-- <BottomNav /> -->
    <!-- Main Content -->
    <div class="p-8 transition-all duration-300" :class="{ 'ml-64': isSidebarOpen }">
      <RouterView />
    </div>
    <!-- <Footer class="" /> -->
  </div>
</template>

<script setup>
import { RouterView } from 'vue-router'
import { ref } from 'vue'
import Navbar from './components/Navbar.vue'
import Sidebar from './components/Sidebar.vue'
// import BottomNav from './components/BottomNav.vue'
import Footer from './components/Footer.vue'
const isSidebarOpen = ref(false)

const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}

const closeSidebar = () => {
  isSidebarOpen.value = false
}
</script>
