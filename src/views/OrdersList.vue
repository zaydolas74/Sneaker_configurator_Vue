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

const deleteOrder = async (orderId: string) => {
  try {
    const response = await fetch(
      `https://sneaker-configurator-api.onrender.com/api/v1/orders/${orderId}`,
      {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );

    const result = await response.json();
    if (response.ok) {
      console.log(`Order deleted successfully: ${result.message}`);
      orders.value = orders.value.filter((o) => o.id !== orderId);
    } else {
      console.error(`Failed to delete order: ${result.message}`);
    }
  } catch (error) {
    console.error("Error deleting order:", error);
  }

  window.location.reload();
};

onMounted(() => {
  fetchOrders();
});
</script>

<template>
  <div class="container mx-auto px-4">
    <h1 class="text-4xl font-extrabold text-gray-800 mb-4">Orders Dashboard</h1>
    <p class="text-lg text-gray-600 mb-6">
      There are
      <span class="font-bold text-gray-800">{{ orders.length }}</span> orders.
    </p>

    <div v-if="loading" class="text-center py-6">
      <div class="text-green-400 font-semibold">Loading orders...</div>
    </div>

    <div v-else-if="error" class="text-center py-6">
      <div class="text-red-600 font-bold">{{ error }}</div>
    </div>

    <div v-else class="relative overflow-hidden rounded-lg">
      <!-- Responsive Scrollable Container -->
      <div class="overflow-x-auto hidden lg:block">
        <table class="min-w-full bg-white">
          <thead class="bg-gray-100">
            <tr>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Customer Email
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Size
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Amount
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Outside Color
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Laces Color
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Sole Color/Material
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Logo
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Date
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Status
              </th>
              <th class="px-6 py-3 text-left text-sm font-medium text-gray-600">
                Action
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="order in orders"
              :key="order.id"
              class="hover:bg-gray-50 transition border-b border-gray-300"
            >
              <td class="px-6 py-4 text-sm text-gray-800">{{ order.email }}</td>
              <td class="px-6 py-4 text-sm text-gray-800">{{ order.size }}</td>
              <td class="px-6 py-4 text-sm text-gray-800">
                {{ order.amount }}
              </td>
              <td class="px-6 py-4 text-sm text-gray-800">
                <div
                  class="w-6 h-6 rounded-full border border-gray-800"
                  :style="{ backgroundColor: order.outsideColor }"
                ></div>
              </td>
              <td class="px-6 py-4 text-sm text-gray-800">
                <div
                  class="w-6 h-6 rounded-full border border-gray-800"
                  :style="{ backgroundColor: order.lacesColor }"
                ></div>
              </td>
              <td class="px-6 py-4 text-sm text-gray-800">
                <div class="flex gap-2">
                  <div
                    class="w-6 h-6 rounded-full border border-gray-800"
                    :style="{ backgroundColor: order.soleColor }"
                  ></div>
                  <span class="font-bold capitalize">
                    {{ order.solesMaterial }}
                  </span>
                </div>
              </td>
              <td class="px-6 py-4 text-sm text-gray-800">
                <img
                  :src="order.logo"
                  alt="Logo"
                  class="w-6 h-6 rounded-full border border-gray-800"
                />
              </td>
              <td class="px-6 py-4 text-sm text-gray-800">{{ order.date }}</td>
              <td class="px-6 py-4 text-sm capitalize">
                <span :class="statusClass(order.status)">
                  {{ order.status }}
                </span>
              </td>
              <td
                class="px-6 py-4 flex flex-col sm:flex-row sm:items-center sm:justify-evenly gap-2"
              >
                <select
                  class="border rounded px-2 py-1 text-gray-700 text-sm"
                  v-model="order.status"
                  @change="updateStatus(order._id, order.status)"
                >
                  <option value="Processing">Processing</option>
                  <option value="Send">Send</option>
                  <option value="Cancelled">Cancelled</option>
                </select>
                <button
                  class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600 transition text-sm"
                  @click="deleteOrder(order._id)"
                >
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <!-- Mobile View -->

      <div class="lg:hidden">
        <div
          v-for="order in orders"
          :key="order.id"
          class="border border-gray-300 bg-white shadow-md rounded-lg p-4 mb-4"
        >
          <div class="mb-2">
            <span class="font-semibold text-gray-600">Email: </span>
            <span class="text-gray-800">{{ order.email }}</span>
          </div>
          <div class="mb-2">
            <span class="font-semibold text-gray-600">Size: </span>
            <span class="text-gray-800">{{ order.size }}</span>
          </div>
          <div class="mb-2">
            <span class="font-semibold text-gray-600">Amount: </span>
            <span class="text-gray-800">{{ order.amount }}</span>
          </div>
          <div class="mb-2 flex items-center">
            <span class="font-semibold text-gray-600">Outside Color: </span>
            <div
              class="w-6 h-6 rounded-full border border-gray-800 ml-2"
              :style="{ backgroundColor: order.outsideColor }"
            ></div>
          </div>
          <div class="mb-2 flex items-center">
            <span class="font-semibold text-gray-600">Laces Color: </span>
            <div
              class="w-6 h-6 rounded-full border border-gray-800 ml-2"
              :style="{ backgroundColor: order.lacesColor }"
            ></div>
          </div>
          <div class="mb-2 flex items-center">
            <span class="font-semibold text-gray-600"
              >Sole Color/Material:
            </span>
            <div class="flex gap-2 ml-2 items-center">
              <div
                class="w-6 h-6 rounded-full border border-gray-800"
                :style="{ backgroundColor: order.soleColor }"
              ></div>
              <span class="font-bold capitalize">{{
                order.solesMaterial
              }}</span>
            </div>
          </div>
          <div class="mb-2 flex items-center">
            <span class="font-semibold text-gray-600">Logo: </span>
            <img
              :src="order.logo"
              alt="Logo"
              class="w-6 h-6 rounded-full border border-gray-800 ml-2"
            />
          </div>
          <div class="mb-2">
            <span class="font-semibold text-gray-600">Date: </span>
            <span class="text-gray-800">{{ order.date }}</span>
          </div>
          <div class="mb-2">
            <span class="font-semibold text-gray-600">Status: </span>
            <span :class="statusClass(order.status)">{{ order.status }}</span>
          </div>
          <div
            class="flex flex-col sm:flex-row sm:items-center sm:justify-evenly gap-2"
          >
            <select
              class="border rounded px-2 py-1 text-gray-700 text-sm"
              v-model="order.status"
              @change="updateStatus(order._id, order.status)"
            >
              <option value="Processing">Processing</option>
              <option value="Send">Send</option>
              <option value="Cancelled">Cancelled</option>
            </select>
            <button
              class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600 transition text-sm"
              @click="deleteOrder(order._id)"
            >
              Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
