<script setup>
import { useRouter } from 'vue-router'
import { isAuthenticated, logout } from '../auth'

const router = useRouter()

const handleLogout = () => {
  logout()
  router.push({ name: 'login' })
}
</script>

<template>
  <nav class="navbar navbar-expand navbar-dark bg-primary">
    <div class="container">
      <router-link class="navbar-brand" to="/">Library</router-link>

      <div class="navbar-nav ms-auto align-items-center">
        <router-link class="nav-link" to="/">Home</router-link>

        <template v-if="isAuthenticated">
          <router-link class="nav-link" to="/about">About</router-link>
          <button class="btn btn-link nav-link" type="button" @click="handleLogout">
            Logout
          </button>
        </template>

        <router-link v-else class="nav-link" to="/login">Login</router-link>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.nav-link {
  color: rgba(255, 255, 255, 0.8);
  margin-left: 0.5rem;
}

.nav-link:hover,
.router-link-active {
  color: white;
  background-color: transparent;
}
</style>
