<template>
  <main class="w-screen h-screen relative bg-white dark:bg-zinc-900 text-dark-50 dark:text-white overflow-hidden"
    @wheel="$event.preventDefault()">
    <router-view></router-view>
    <!-- <Navigation></Navigation> -->
    <Settings></Settings>
  </main>
  <Loading v-if="isFetching"></Loading>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const isFetching = ref(false)
</script>
