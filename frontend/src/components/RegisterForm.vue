<template>
  <form class="form" @submit.prevent="submitRegister">
    <label>
      Name
      <input type="text" v-model="name" required />
    </label>
    <label>
      Email
      <input type="email" v-model="email" required />
    </label>
    <label>
      Password
      <input type="password" v-model="password" required minlength="6" />
    </label>
    <button type="submit">Register</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['result'])
const name = ref('')
const email = ref('')
const password = ref('')
const apiUrl = import.meta.env.VITE_API_URL || '/api'

async function submitRegister() {
  try {
    const response = await fetch(`${apiUrl}/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: name.value, email: email.value, password: password.value })
    })

    const data = await response.json()
    if (!response.ok) throw new Error(data.message || 'Register failed')

    emit('result', {
      message: 'Registered successfully. Token saved.',
      token: data.token
    })
  } catch (error) {
    emit('result', { message: error.message })
  }
}
</script>
