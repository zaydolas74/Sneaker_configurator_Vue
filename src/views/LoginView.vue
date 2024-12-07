<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100">
    <div class="w-full max-w-md bg-white p-6 rounded-lg shadow-md">
      <h1 class="text-2xl font-bold text-center text-gray-800 mb-6">Login</h1>
      <form @submit.prevent="handleLogin" class="space-y-4">
        <div>
          <label for="username" class="block text-sm font-medium text-gray-700"
            >Username</label
          >
          <input
            id="username"
            v-model="username"
            type="text"
            placeholder="Enter your username"
            class="mt-1 w-full px-4 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-green-400 focus:border-green-400"
          />
        </div>
        <div>
          <label for="password" class="block text-sm font-medium text-gray-700"
            >Password</label
          >
          <input
            id="password"
            v-model="password"
            type="password"
            placeholder="Enter your password"
            class="mt-1 w-full px-4 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-green-400 focus:border-green-400"
          />
        </div>
        <p v-if="errorMessage" class="text-sm text-red-400">
          {{ errorMessage }}
        </p>
        <button
          type="submit"
          class="w-full py-2 px-4 bg-green-400 text-white font-medium rounded-md hover:bg-green-500 focus:outline-none focus:ring-2 focus:ring-green-400 focus:ring-offset-2"
        >
          Login
        </button>
      </form>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import { useRouter } from "vue-router";

export default {
  setup() {
    const username = ref("");
    const password = ref("");
    const errorMessage = ref("");
    const router = useRouter();

    const handleLogin = async () => {
      if (!username.value || !password.value) {
        errorMessage.value = "Please enter both username and password.";
        return;
      }

      try {
        const response = await fetch(
          "https://sneaker-configurator-api.onrender.com/api/v1/users/login",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              username: username.value,
              password: password.value,
            }),
          }
        );

        const data = await response.json();

        console.log(data);

        if (data.status === "success" && data.token) {
          localStorage.setItem("token", data.token);
          router.push("/home");
        } else {
          errorMessage.value = "Invalid login credentials.";
        }
      } catch (error) {
        errorMessage.value = "An error occurred. Please try again later.";
        console.error("error");
      }
    };

    return { username, password, errorMessage, handleLogin };
  },
};
</script>
