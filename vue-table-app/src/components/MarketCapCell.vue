<template>
  <span :class="{
    'text-green-600': marketCap >= 1000000000,
    'text-blue-600': marketCap >= 100000000 && marketCap < 1000000000,
    'text-gray-600': marketCap < 100000000
  }">
    {{ formattedMarketCap }}
  </span>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: {
    type: [Number, String],
    required: true
  }
})

const marketCap = computed(() => {
  const val = Number(props.value)
  return isNaN(val) ? 0 : val
})

const formattedMarketCap = computed(() => {
  const value = marketCap.value
  
  if (value >= 1000000000) {
    return `$${(value / 1000000000).toFixed(1)}B`
  } else if (value >= 1000000) {
    return `$${(value / 1000000000).toFixed(2)}B`
  } else if (value >= 1000) {
    return `$${(value / 1000000).toFixed(2)}M`
  }
  
  return `$${value.toLocaleString()}`
})
</script>