<template>
  <div class="relative">
    <input
      v-model="filterValue"
      @input="handleInput"
      :placeholder="placeholder"
      class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-blue-500 focus:border-blue-500 text-sm"
      type="text"
    />
    <button
      v-if="filterValue"
      @click="clearFilter"
      class="absolute right-2 top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600"
      type="button"
      title="Очистить фильтр"
    >
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
      </svg>
    </button>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  placeholder: {
    type: String,
    default: 'Фильтр...'
  },
  column: {
    type: String,
    required: true
  },
  modelValue: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['update:modelValue', 'filter'])

// Используем ref для локального значения
const filterValue = ref(props.modelValue)

// Отслеживаем изменения modelValue извне
watch(() => props.modelValue, (newValue) => {
  filterValue.value = newValue
})

const handleInput = () => {
  // Эмитим обновление v-model
  emit('update:modelValue', filterValue.value)
  // Эмитим событие фильтрации
  emit('filter', { 
    key: props.column, 
    value: filterValue.value 
  })
}

const clearFilter = () => {
  filterValue.value = ''
  emit('update:modelValue', '')
  emit('filter', { 
    key: props.column, 
    value: '' 
  })
}
</script>