<template>
  <div class="recaptcha-view">
    <nav class="recaptcha-nav">
      <RouterLink to="/">← Volver al Home</RouterLink>
    </nav>
    <RecaptchaDemo />
  </div>
</template>

<script setup lang="ts">
import { RouterLink } from 'vue-router'
import RecaptchaDemo from '@/components/RecaptchaDemo.vue'
</script>

<style scoped>
.recaptcha-view {
  min-height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 0;
  margin: 0;
  overflow-x: hidden;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

.recaptcha-nav {
  background: #2c3e50;
  padding: 15px 20px;
  text-align: center;
}

.recaptcha-nav a {
  color: white;
  text-decoration: none;
  font-weight: bold;
  padding: 10px 20px;
  border-radius: 5px;
  background: #3498db;
  transition: background 0.3s;
}

.recaptcha-nav a:hover {
  background: #2980b9;
}
</style>
