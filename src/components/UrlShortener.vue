<template>
  <Card>
    <CardHeader>
      <CardTitle>Elongate Your URL</CardTitle>
      <CardDescription> I haven't bought a custom domain so there's a decent chance your URL will get longer.<br>
        Enter a URL below to create an elongated version </CardDescription>
    </CardHeader>

    <CardContent class="space-y-4">
      <form @submit.prevent="shortenUrl" class="flex items-center space-x-4">
        <Input
          v-model="originalUrl"
          type="url"
          placeholder="https://example.com/"
          :disabled="loading"
          class="flex-1"
        />
        <Button type="submit" :disabled="loading || !originalUrl">
          <Loader2 v-if="loading" class="w-4 h-4 mr-2 animate-spin" />
          {{ loading ? 'Changing...' : 'Change' }}
        </Button>
      </form>

      <Alert v-if="error" variant="destructive">
        <AlertCircle class="h-4 w-4" />
        <AlertTitle>Error</AlertTitle>
        <AlertDescription>{{ error }}</AlertDescription>
      </Alert>

      <Card v-if="result" class="border-green-200 bg-green-50">
        <CardHeader>
          <CardTitle class="text-green-800 flex items-center gap-2">
            <CheckCircle class="h-5 w-5" />
            URL Elongated Successfully!
          </CardTitle>
        </CardHeader>
        <CardContent class="space-y-4">
          <div class="space-y-3">
            <Input :modelValue="result.short_url" readonly ref="resultInput" />
            <Button @click="copyToClipboard" variant="outline" size="sm" class="w-full">
              <Copy v-if="!copied" class="w-4 h-4 mr-2" />
              <Check v-else class="w-4 h-4 mr-2" />
              {{ copied ? 'Copied!' : 'Copy' }}
            </Button>
          </div>
        </CardContent>
      </Card>
    </CardContent>
  </Card>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { Loader2, AlertCircle, CheckCircle, Copy, Check } from 'lucide-vue-next'

interface ShortenResponse {
  short_url: string
  short_code: string
}

const emit = defineEmits<{
  'url-shortened': [shortCode: string]
}>()

const originalUrl = ref('')
const loading = ref(false)
const error = ref('')
const result = ref<ShortenResponse | null>(null)
const copied = ref(false)
const resultInput = ref<HTMLInputElement>()

const API_BASE = import.meta.env.VITE_API_URL || 'http://localhost:5000'

const shortenUrl = async () => {
  if (!originalUrl.value) return

  loading.value = true
  error.value = ''
  result.value = null
  copied.value = false

  try {
    const response = await axios.post<ShortenResponse>(`${API_BASE}/api/shorten`, {
      url: originalUrl.value,
    })

    result.value = response.data
    emit('url-shortened', response.data.short_code)
  } catch (err: unknown) {
    if (axios.isAxiosError(err) && err.response?.data?.error) {
      error.value = err.response.data.error
    } else {
      error.value = 'Failed to shorten URL'
    }
  } finally {
    loading.value = false
  }
}

const copyToClipboard = async () => {
  if (!result.value) return

  try {
    await navigator.clipboard.writeText(result.value.short_url)
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch {
    // Fallback for older browsers
    if (resultInput.value) {
      resultInput.value.select()
      document.execCommand('copy')
      copied.value = true
      setTimeout(() => {
        copied.value = false
      }, 2000)
    }
  }
}
</script>
