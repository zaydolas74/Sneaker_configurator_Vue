<script setup lang="ts">
import { ref } from "vue";
import { useRouter } from "vue-router";
import {
  KeyIcon,
  ClipboardIcon,
  FingerPrintIcon,
} from "@heroicons/vue/24/outline";
const router = useRouter();

const navigation = [
  { name: "Orders", href: "/home/", icon: ClipboardIcon },
  { name: "Change Password", href: "/home/change-password", icon: KeyIcon },
];

const logout = () => {
  localStorage.removeItem("token");
  window.location.href = "/";
};
</script>

<template>
  <div class="flex h-screen flex-col justify-between border-r bg-white">
    <div class="px-4 py-6">
      <div class="flex items-center gap-2 mb-8">
        <img src="../assets/logo.png" />
      </div>

      <nav aria-label="Main Nav" class="mt-6 flex flex-col space-y-1">
        <router-link
          v-for="item in navigation"
          :key="item.name"
          :to="item.href"
          class="flex items-center gap-2 rounded-lg px-4 py-2 text-gray-700 hover:bg-gray-100 hover:text-gray-700"
          :class="{ 'bg-gray-100': $route.path === item.href }"
        >
          <component :is="item.icon" class="h-5 w-5" />
          <span>{{ item.name }}</span>
        </router-link>
        <button
          @click="logout"
          class="flex items-center gap-2 rounded-lg px-4 py-2 text-gray-700 hover:bg-gray-100 hover:text-gray-700"
        >
          <FingerPrintIcon class="h-5 w-5" />

          <span>Logout</span>
        </button>
      </nav>
    </div>
  </div>
</template>
