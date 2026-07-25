<template>
  <form class="form" @submit.prevent="submitLogin">
    <label>
      Email
      <input type="email" v-model="email" required />
    </label>
    <label>
      Password
      <input type="password" v-model="password" required />
    </label>
    <button type="submit">Login</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['result'])
const email = ref('')
const password = ref('')
const apiUrl = import.meta.env.VITE_API_URL || '/api'

async function submitLogin() {
  try {
    const response = await fetch(`${apiUrl}/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email.value, password: password.value })
    })

    const data = await response.json()
    if (!response.ok) throw new Error(data.message || 'Login failed')

    emit('result', {
      message: 'Login successful. Token saved.',
      token: data.token
    })
  } catch (error) {
    emit('result', { message: error.message })
  }
}
</script>
