<script setup lang="ts">
import Sidebar from "../components/Sidebar.vue";
</script>

<template>
  <div class="flex h-screen bg-gray-50">
    <!-- Sidebar -->
    <Sidebar class="w-64 flex-shrink-0" />

    <!-- Main Content Area -->
    <div class="flex-1 overflow-auto p-8">
      <router-view></router-view>
    </div>
  </div>
</template>
