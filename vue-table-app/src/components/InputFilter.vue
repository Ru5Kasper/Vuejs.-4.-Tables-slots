<template>
  <div class="relative">
    <input
      :value="inputValue"
      @input="handleInput"
      :placeholder="placeholder"
      class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-blue-500 focus:border-blue-500 text-sm"
      type="text"
    />
    <button
      v-if="inputValue"
      @click="clearFilter"
      class="absolute right-2 top-1/2 transform -translate-y-1/2 text-gray-400 hover:text-gray-600 transition-colors duration-150"
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
import { ref, watch, onUnmounted } from 'vue'
import debounce from 'lodash.debounce'

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

const inputValue = ref(props.modelValue)

// Дебаунсинг для ввода (100мс)
const emitFilterDebounced = debounce((value) => {
  emit('update:modelValue', value)
  emit('filter', { 
    key: props.column, 
    value: value 
  })
}, 100)

const handleInput = (event) => {
  const value = event.target.value
  inputValue.value = value
  emitFilterDebounced(value)
}

const clearFilter = () => {
  inputValue.value = ''
  emitFilterDebounced.cancel() // Отменяем предыдущий вызов
  emit('update:modelValue', '')
  emit('filter', { 
    key: props.column, 
    value: '' 
  })
}

// Отслеживаем изменения modelValue извне
watch(() => props.modelValue, (newValue) => {
  inputValue.value = newValue
})

// Очищаем таймер при уничтожении компонента
onUnmounted(() => {
  emitFilterDebounced.cancel()
})
</script>