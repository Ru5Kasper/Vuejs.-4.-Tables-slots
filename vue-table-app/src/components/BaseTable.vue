<template>
  <div class="overflow-x-auto bg-white rounded-lg shadow">
    <table class="min-w-full divide-y divide-gray-200">
      <thead class="bg-gray-50">
        <tr>
          <th 
            v-for="(headerText, key) in headers" 
            :key="key"
            class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
          >
            <!-- Слот для кастомного заголовка -->
            <div class="space-y-2">
              <slot 
                v-if="$slots[`header-${key}`]" 
                :name="`header-${key}`" 
                :key="key" 
                :text="headerText"
              >
                {{ headerText }}
              </slot>
              <div v-else>
                {{ headerText }}
              </div>
              
              <!-- Кнопка очистки фильтра для этой колонки -->
              <button
                v-if="activeFilters[key]"
                @click="clearColumnFilter(key)"
                class="mt-1 text-xs text-gray-400 hover:text-gray-600 flex items-center gap-1"
                title="Очистить фильтр"
              >
                <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
                Очистить
              </button>
            </div>
          </th>
        </tr>
      </thead>
      <tbody class="bg-white divide-y divide-gray-200">
        <tr 
          v-for="(row, rowIndex) in rows" 
          :key="rowIndex"
          class="hover:bg-gray-50"
        >
          <td 
            v-for="(value, key) in row" 
            :key="key"
            class="px-6 py-4 whitespace-nowrap"
          >
            <!-- Динамический слот для ячейки -->
            <slot 
              v-if="$slots[key]" 
              :name="key" 
              :value="value" 
              :row="row"
            >
              {{ value }}
            </slot>
            <span v-else>{{ value }}</span>
          </td>
        </tr>
      </tbody>
    </table>
    
    <!-- Панель управления фильтрами -->
    <div v-if="hasActiveFilters" class="px-6 py-3 bg-gray-50 border-t border-gray-200 flex items-center justify-between">
      <span class="text-sm text-gray-500">
        Активные фильтры: {{ activeFilterCount }}
      </span>
      <button
        @click="clearAllFilters"
        class="px-4 py-2 text-sm font-medium text-red-600 hover:text-red-800"
      >
        Очистить все фильтры
      </button>
    </div>
    
    <!-- Сообщение, если нет данных -->
    <div v-if="rows.length === 0" class="px-6 py-12 text-center text-gray-500">
      Нет данных для отображения
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  rows: {
    type: Array,
    required: true,
    default: () => []
  },
  headers: {
    type: Object,
    required: true,
    default: () => ({})
  },
  activeFilters: {
    type: Object,
    default: () => ({})
  }
})

const emit = defineEmits(['filter-change', 'clear-filter'])

// Вычисляемые свойства
const hasActiveFilters = computed(() => Object.keys(props.activeFilters).length > 0)
const activeFilterCount = computed(() => Object.keys(props.activeFilters).length)

// Методы
const clearColumnFilter = (key) => {
  emit('clear-filter', key)
}

const clearAllFilters = () => {
  emit('filter-change', {})
}
</script>