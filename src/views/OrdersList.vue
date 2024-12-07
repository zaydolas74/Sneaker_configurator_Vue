<script setup lang="ts">
import { ref, onMounted } from "vue";

const orders = ref([]);
const loading = ref(true);
const error = ref(null);

const fetchOrders = async () => {
  try {
    const response = await fetch(
      "https://sneaker-configurator-api.onrender.com/api/v1/orders",
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    const data = await response.json();
    if (data.status === "success") {
      orders.value = data.data.orders;
    } else {
      error.value = "Failed to fetch orders";
    }
  } catch (err) {
    error.value = "Error fetching orders";
  } finally {
    loading.value = false;
  }
};

const updateStatus = async (orderId: string, newStatus: string) => {
  try {
    const response = await fetch(
      `https://sneaker-configurator-api.onrender.com/api/v1/orders/${orderId}`,
      {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
        body: JSON.stringify({ status: newStatus }),
      }
    );

    const result = await response.json();
    if (response.ok) {
      console.log(`Order status updated successfully: ${result.message}`);
      // Optioneel: status lokaal updaten om herladen te vermijden
      const order = orders.value.find((o) => o.id === orderId);
      if (order) order.status = newStatus;
    } else {
      console.error(`Failed to update order status: ${result.message}`);
    }
  } catch (error) {
    console.error("Error updating order status:", error);
  }
};

const statusClass = (status: string) => {
  switch (status) {
    case "Processing":
      return "text-blue-500 font-semibold";
    case "Send":
      return "text-green-500 font-semibold";
    case "Cancelled":
      return "text-red-500 font-semibold";
    default:
      return "text-gray-500 font-semibold";
  }
};

onMounted(() => {
  fetchOrders();
});
</script>

<template>
  <div class="container mx-auto p-6">
    <h1 class="text-4xl font-extrabold text-gray-800 mb-4">Orders Dashboard</h1>
    <p class="text-lg text-gray-600 mb-6">
      You have
      <span class="font-bold text-gray-800">{{ orders.length }}</span> orders.
    </p>

    <div v-if="loading" class="text-center py-6">
      <div class="text-green-400 font-semibold">Loading orders...</div>
    </div>

    <div v-else-if="error" class="text-center py-6">
      <div class="text-red-600 font-bold">{{ error }}</div>
    </div>

    <div v-else>
      <table
        class="w-full border-collapse border border-gray-200 bg-white rounded-lg shadow-md"
      >
        <thead class="bg-gray-100">
          <tr>
            <th class="border border-gray-300 p-4 text-left">Customer Email</th>
            <th class="border border-gray-300 p-4 text-left">Date</th>
            <th class="border border-gray-300 p-4 text-left">Status</th>
            <th class="border border-gray-300 p-4 text-left">Action</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="order in orders"
            :key="order.id"
            class="hover:bg-gray-50 transition"
          >
            <td class="border border-gray-300 p-4">{{ order.email }}</td>
            <td class="border border-gray-300 p-4">{{ order.date }}</td>
            <td class="border border-gray-300 p-4">
              <span :class="statusClass(order.status)">
                {{ order.status }}
              </span>
            </td>
            <td class="border border-gray-300 p-4">
              <select
                class="border rounded px-2 py-1 text-gray-700"
                v-model="order.status"
                @change="updateStatus(order._id, order.status)"
              >
                <option value="Processing">Processing</option>
                <option value="Send">Send</option>
                <option value="Cancelled">Cancelled</option>
              </select>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
