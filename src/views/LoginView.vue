<script setup>
import router from '@/router'
import { ref } from 'vue'

const username = ref('')
const password = ref('')

const onSubmit = async (e) => {
  e.preventDefault()
  try {
    const res = await fetch('http://localhost:3000/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        username: username.value,
        password: password.value,
      }),
    })

    const data = await res.json()

    if (res.ok) {
      alert('Login successful!')
      console.log('Login response data:', data)
      router.push({ name: 'Home' })
    } else {
      console.log('Login failed response data:', data)
      const errorMessage = data.message || data.error || 'Unknown error'
      alert('Login failed: ' + errorMessage)
    }
  } catch (e) {
    console.error('Error during login', e)
    alert('Error during login: ' + e.message)
  }
}

</script>
<template>
  <div class="flex flex-col items-center justify-center h-screen">
    <h1 class="text-3xl mb-6">Login Page</h1>
    <el-form class="flex flex-col w-80">
      <el-form-item>
        <el-input v-model="username" type="text" placeholder="Username" clearable />
      </el-form-item>
      <el-form-item>
        <el-input v-model="password" type="password" placeholder="Password" clearable />
      </el-form-item>
      <el-form-item>
        <el-button class="w-full" @click="onSubmit">Login</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
