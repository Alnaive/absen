<template>
  <div class="navbar text-black dark:text-white bg-white dark:bg-base-100 shadow-lg">
    <!-- <div class="flex-none invisible sm:visible">
      <button @click="toggleSidebar" class="btn btn-square btn-ghost">
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
    </div> -->
    <div class="flex-1">
      <a class="btn btn-ghost text-xl">KLS</a>
    </div>
    <div class="flex-none">
     <button @click="toggleDark()">
        <Icon icon="bi:moon-fill" width="24" height="24" v-if="isDark"/> 
        <Icon icon="akar-icons:sun-fill" width="28" height="28" v-else/>
      </button>
    </div>
  </div>
</template>

<script setup>
import { useDark, useToggle } from '@vueuse/core';
import { Icon } from '@iconify/vue';

const props = defineProps({
  isSidebarOpen: {
    type: Boolean,
    required: true,
  },
})

const emit = defineEmits(['toggleSidebar'])

const toggleSidebar = () => {
  emit('toggleSidebar')
}

const isDark = useDark();
const toggleDark = useToggle(isDark);
</script>
