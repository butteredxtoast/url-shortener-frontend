<template>
  <div id="app" class="min-h-screen bg-background">
    <main class="container mx-auto px-4 py-8 max-w-4xl">
      <UrlShortener @url-shortened="handleUrlShortened" />
      <UrlStats v-if="lastShortenedCode" :shortCode="lastShortenedCode" class="mt-8" />
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import UrlShortener from './components/UrlShortener.vue'
import UrlStats from './components/UrlStats.vue'

const lastShortenedCode = ref<string>('')

const handleUrlShortened = (shortCode: string) => {
  lastShortenedCode.value = shortCode
}
</script>
