<template>
  <div class="flex w-full p-4 flex-col items-center justify-between">
    <div class="flex flex-col items-center">
      <h1 class="text-xl font-bold mb-4">DHCP Users</h1>

      <button
          class="bg-blue-500 text-white px-4 py-2 rounded mb-4 hover:bg-blue-600 transition-colors"
          @click="makeUser"
          :disabled="loading"
      >
        {{ loading ? 'Creating...' : 'Make a New User' }}
      </button>
    </div>

    <div v-if="users.length === 0" class="text-gray-500 font-bold">
      No users created yet. Click the button to create your first user.
    </div>

  <div>
    <div v-if="error" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
      {{ error }}
    </div>



    <div class="w-[80%] grid grid-cols-1 mx-auto border shadow">
      <div class="grid space-x-16 grid-cols-5 w-full border-b bg-gray-200 pr-6 pl-4">
        <span>User</span>
        <span>MAC</span>
        <span>IP</span>
        <span>Status</span>
        <span>delete</span>
      </div>

      <div v-for="user in users" :key="user._id" class="grid grid-cols-5 space-x-16 border-b px-2 py-1">
        <span>{{user.name}}</span>
        <span>{{user.mac}}</span>
        <span>{{user.ip || 'not assigned'}}</span>
        <span>{{user.status}}</span>
        <span class="cursor-pointer text-red-600" @click="deleteUser(user._id)">delete</span>
      </div>
    </div>
  </div>


  </div>
</template>

<script setup>
import {ref, onMounted} from 'vue';
import axios from 'axios';

const users = ref([]);
const loading = ref(false);
const error = ref('');

const fetchUsers = async () => {
  try {
    const res = await axios.get('http://localhost:8080/clients');
    users.value = res.data;
  } catch (err) {
    error.value = 'Failed to fetch users: ' + err.message;
    console.error(err);
  }
};

const makeUser = async () => {
  loading.value = true;
  error.value = '';
  try {
    const res = await axios.post('http://localhost:8080/make-user');
    users.value.push(res.data);
  } catch (err) {
    error.value = 'Failed to create user: ' + err.message;
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const deleteUser = async (userId) => {
  if (!confirm('Are you sure you want to delete this user?')) return;

  try {
    await axios.delete(`http://localhost:8080/users/${userId}`);
    users.value = users.value.filter(user => user._id !== userId);
  } catch (err) {
    error.value = 'Failed to delete user: ' + err.message;
    console.error(err);
  }
};

onMounted(() => {
  fetchUsers();
});
</script>

<style scoped>
</style>