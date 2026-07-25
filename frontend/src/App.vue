<template>
  <div class="app-shell">
    <div class="card">
      <h1>Laravel + Vue Auth</h1>
      <p class="subtitle">Use the form below to register or login.</p>

      <div class="tabs">
        <button :class="{ active: mode === 'login' }" @click="mode = 'login'">Login</button>
        <button :class="{ active: mode === 'register' }" @click="mode = 'register'">Register</button>
      </div>

      <LoginForm v-if="mode === 'login'" @result="handleResult" />
      <RegisterForm v-else @result="handleResult" />

      <div v-if="message" class="message">{{ message }}</div>

      <div v-if="token" class="profile-card">
        <h2>Authenticated</h2>
        <p><strong>Token:</strong> {{ token }}</p>
        <div class="button-row">
          <button @click="fetchProfile">Load Profile</button>
          <button @click="logout" class="logout">Logout</button>
        </div>

        <div v-if="profile" class="profile">
          <h3>Profile</h3>
          <p><strong>Name:</strong> {{ profile.name }}</p>
          <p><strong>Email:</strong> {{ profile.email }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import LoginForm from './components/LoginForm.vue'
import RegisterForm from './components/RegisterForm.vue'

const mode = ref('login')
const message = ref('')
const token = ref('')
const profile = ref(null)
const apiUrl = import.meta.env.VITE_API_URL || '/api'

onMounted(() => {
  const saved = localStorage.getItem('auth_token')
  if (saved) {
    token.value = saved
    fetchProfile()
  }
})

function handleResult(payload) {
  const result = payload?.detail ?? payload
  if (result?.token) {
    token.value = result.token
    localStorage.setItem('auth_token', result.token)
    message.value = result.message
    fetchProfile()
    return
  }

  message.value = result?.message ?? String(result)
}

async function fetchProfile() {
  if (!token.value) {
    message.value = 'No auth token available.'
    return
  }

  message.value = ''
  try {
    const response = await fetch(`${apiUrl}/profile`, {
      headers: {
        Authorization: `Bearer ${token.value}`
      }
    })

    const data = await response.json()
    if (!response.ok) throw new Error(data.message || 'Failed loading profile')
    profile.value = data.user
  } catch (error) {
    profile.value = null
    message.value = error.message
  }
}

function logout() {
  token.value = ''
  profile.value = null
  message.value = 'Logged out'
  localStorage.removeItem('auth_token')
}
</script>
