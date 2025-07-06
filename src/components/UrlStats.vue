<template>
  <Card v-if="stats">
    <CardHeader>
      <CardTitle class="flex items-center gap-2">
        <BarChart3 class="h-5 w-5" />
        URL Statistics
      </CardTitle>
      <CardDescription> Analytics for your shortened URL </CardDescription>
    </CardHeader>

    <CardContent class="space-y-4">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="space-y-2">
          <div class="text-sm font-medium text-muted-foreground">Original URL</div>
          <div class="text-sm break-all bg-muted p-2 rounded">
            {{ stats.original_url }}
          </div>
        </div>

        <div class="space-y-2">
          <div class="text-sm font-medium text-muted-foreground">Short Code</div>
          <Badge variant="secondary" class="font-mono">
            {{ stats.short_code }}
          </Badge>
        </div>
      </div>

      <Separator />

      <div class="grid grid-cols-2 gap-4">
        <div class="text-center">
          <div class="text-2xl font-bold text-primary">{{ stats.clicks }}</div>
          <div class="text-sm text-muted-foreground">Total Clicks</div>
        </div>

        <div class="text-center">
          <div class="text-sm font-medium">{{ formatDate(stats.created_at) }}</div>
          <div class="text-sm text-muted-foreground">Created</div>
        </div>
      </div>
    </CardContent>
  </Card>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import axios from 'axios'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Separator } from '@/components/ui/separator'
import { BarChart3 } from 'lucide-vue-next'

interface UrlStats {
  short_code: string
  original_url: string
  clicks: number
  created_at: string
}

const props = defineProps<{
  shortCode: string
}>()

const stats = ref<UrlStats | null>(null)
const API_BASE = import.meta.env.VITE_API_URL || 'http://localhost:5000'

const fetchStats = async () => {
  if (!props.shortCode) return

  try {
    const response = await axios.get<UrlStats>(`${API_BASE}/api/stats/${props.shortCode}`)
    stats.value = response.data
  } catch (err) {
    console.error('Failed to fetch stats:', err)
  }
}

const formatDate = (dateString: string) => {
  return new Date(dateString).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  })
}

watch(() => props.shortCode, fetchStats, { immediate: true })
</script>
