<script setup>
import { RouterLink, RouterView } from 'vue-router';
</script>

<template>
  <header>
    <div>
      <nav>
        <ul>
          <li>
            <button class="btn" @click="destination = ''">
              <RouterLink to="/">Home</RouterLink>
            </button>
          </li>
        </ul>
      </nav>
    </div>
  </header>
  <RouterView />
</template>

<style scoped></style>
