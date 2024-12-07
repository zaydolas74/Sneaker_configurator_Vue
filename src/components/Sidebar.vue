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
  { name: "Orders", href: "/home", icon: ClipboardIcon },
  { name: "Change Password", href: "/home/change-password", icon: KeyIcon },
];

const logout = () => {
  localStorage.removeItem("token");
  window.location.href = "/";
};

const currentPath = ref(window.location.pathname); // Track the current path

// Watch for route changes to update the currentPath
router.afterEach((to) => {
  currentPath.value = to.path;
});
</script>

<template>
  <div
    class="flex h-screen flex-col justify-between border-r bg-white hidden w-64 sm:block"
  >
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

  <!-- Mobile Navigation -->
  <nav aria-label="bottom nav" class="sm:hidden">
    <div class="bg-neutral-900 shadow-xl h-16 fixed bottom-0 w-full z-10">
      <div class="flex justify-between px-4 py-2">
        <ul class="flex w-full justify-around">
          <li>
            <a
              href="#"
              class="block py-1 text-white hover:text-gray-400 flex gap-2 items-center flex-col"
              @click="router.push('/home')"
            >
              <ClipboardIcon class="h-5 w-5" />
              <span class="text-xs">Order List</span>
            </a>
          </li>
          <li>
            <a
              href="#"
              class="block py-1 text-white hover:text-gray-400 flex gap-2 items-center flex-col"
              @click="router.push('/home/change-password')"
            >
              <KeyIcon class="h-5 w-5" />
              <span class="text-xs">Change Password</span>
            </a>
          </li>
          <li>
            <a
              href="#"
              @click="logout"
              class="block py-1 text-white hover:text-gray-400 flex gap-2 items-center flex-col"
            >
              <FingerPrintIcon class="h-5 w-5" />
              <span class="text-xs">Logout</span>
            </a>
          </li>
        </ul>
      </div>
    </div>
  </nav>
</template>
