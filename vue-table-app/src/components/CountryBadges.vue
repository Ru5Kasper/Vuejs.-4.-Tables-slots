<template>
  <div class="flex flex-wrap gap-1">
    <span
      v-for="(country, index) in countries"
      :key="index"
      class="inline-flex items-center px-2 py-1 rounded-full text-xs font-medium bg-blue-100 text-blue-800"
    >
      {{ country }}
    </span>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: {
    type: [Array, String],
    required: true
  }
})

const countries = computed(() => {
  return Array.isArray(props.value) ? props.value : [props.value]
})
</script>