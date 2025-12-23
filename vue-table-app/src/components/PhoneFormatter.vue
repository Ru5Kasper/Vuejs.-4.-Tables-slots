<template>
  <a 
    :href="`tel:${formattedPhone}`"
    class="text-blue-600 hover:text-blue-800 hover:underline"
  >
    {{ formattedPhone }}
  </a>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  value: {
    type: String,
    required: true
  }
})

const formattedPhone = computed(() => {
  const phone = props.value
  
  if (!phone) return ''
  
  // Удаляем все нецифровые символы
  const digits = phone.replace(/\D/g, '')
  
  // Если номер начинается с 7 или 8, добавляем +7
  if (digits.startsWith('7') && digits.length === 11) {
    return `+7 ${digits.slice(1, 4)} ${digits.slice(4, 7)} ${digits.slice(7, 9)} ${digits.slice(9)}`
  } else if (digits.startsWith('8') && digits.length === 11) {
    return `+7 ${digits.slice(1, 4)} ${digits.slice(4, 7)} ${digits.slice(7, 9)} ${digits.slice(9)}`
  } else if (digits.length === 10) {
    return `+7 ${digits.slice(0, 3)} ${digits.slice(3, 6)} ${digits.slice(6, 8)} ${digits.slice(8)}`
  }
  
  return phone
})
</script>