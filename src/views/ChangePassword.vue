<script setup lang="ts">
import { ref } from "vue";

const oldPassword = ref("");
const newPassword = ref("");
const confirmPassword = ref("");
const error = ref("");
const success = ref("");
const loading = ref(false);

const handleChangePassword = async () => {
  if (newPassword.value !== confirmPassword.value) {
    error.value = "New passwords do not match";
    return;
  }

  if (newPassword.value.length < 8) {
    error.value = "New password must be at least 8 characters long";
    return;
  }

  loading.value = true;
  error.value = "";
  success.value = "";

  try {
    const response = await fetch(
      "https://sneaker-configurator-api.onrender.com/api/v1/users/change-password",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
        body: JSON.stringify({
          oldPassword: oldPassword.value,
          newPassword: newPassword.value,
          confirmPassword: newPassword.value,
        }),
      }
    );

    const data = await response.json();
    if (data.status === "success") {
      success.value = "Password changed successfully";
      oldPassword.value = "";
      newPassword.value = "";
      confirmPassword.value = "";
    } else {
      error.value = data.message || "Failed to change password";
    }
  } catch (err) {
    error.value = "An error occurred. Please try again." + err;
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <div class="max-w-md mx-auto p-6 bg-white shadow-md rounded-lg">
    <h2 class="text-2xl font-bold text-gray-900 mb-4">Change Password</h2>

    <!-- Foutmelding -->
    <div v-if="error" class="text-red-500 mb-4">{{ error }}</div>

    <!-- Succesmelding -->
    <div v-if="success" class="text-green-500 mb-4">{{ success }}</div>

    <!-- Formulier -->
    <form @submit.prevent="handleChangePassword">
      <div class="mb-4">
        <label for="oldPassword" class="block text-gray-700"
          >Old Password</label
        >
        <input
          type="password"
          id="oldPassword"
          v-model="oldPassword"
          class="mt-1 p-2 w-full border border-gray-300 rounded-md"
          required
        />
      </div>

      <div class="mb-4">
        <label for="newPassword" class="block text-gray-700"
          >New Password</label
        >
        <input
          type="password"
          id="newPassword"
          v-model="newPassword"
          class="mt-1 p-2 w-full border border-gray-300 rounded-md"
          required
        />
      </div>

      <div class="mb-4">
        <label for="confirmPassword" class="block text-gray-700"
          >Confirm New Password</label
        >
        <input
          type="password"
          id="confirmPassword"
          v-model="confirmPassword"
          class="mt-1 p-2 w-full border border-gray-300 rounded-md"
          required
        />
      </div>

      <button
        type="submit"
        class="w-full py-2 px-4 bg-green-400 text-white rounded-md hover:bg-green-500"
      >
        Change Password
      </button>
    </form>
  </div>
</template>
