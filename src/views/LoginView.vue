<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { login } from '../auth'

const username = ref('')
const password = ref('')
const router = useRouter()

const submitLogin = () => {
  if (login(username.value, password.value)) {
    router.push({ name: 'about' })
  } else {
    router.push({ name: 'access-denied' })
  }
}
</script>

<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-md-6">
        <h1 class="text-center mb-4">Login</h1>

        <form @submit.prevent="submitLogin">
          <div class="mb-3">
            <label for="loginUsername" class="form-label">Username</label>
            <input
              id="loginUsername"
              v-model="username"
              type="text"
              class="form-control"
              required
            >
          </div>

          <div class="mb-3">
            <label for="loginPassword" class="form-label">Password</label>
            <input
              id="loginPassword"
              v-model="password"
              type="password"
              class="form-control"
              required
            >
          </div>

          <button type="submit" class="btn btn-primary w-100">Login</button>
        </form>
      </div>
    </div>
  </div>
</template>
