<template>
  <div class="min-h-screen bg-gray-100 p-4 md:p-8">
    <div class="max-w-7xl mx-auto">
      <header class="mb-8">
        <h1 class="text-3xl font-bold text-gray-900">Таблица пользователей</h1>
        <p class="mt-2 text-gray-600">
          Демонстрация динамических слотов и фильтрации в Vue.js
        </p>
      </header>

      <div class="bg-white rounded-lg shadow-lg p-6">
        <!-- Панель управления фильтрами -->
        <div class="mb-6 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Фильтр по имени</label>
            <input
              v-model="filters.firstName"
              type="text"
              placeholder="Введите имя..."
              class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Фильтр по фамилии</label>
            <input
              v-model="filters.lastName"
              type="text"
              placeholder="Введите фамилию..."
              class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Фильтр по email</label>
            <input
              v-model="filters.email"
              type="text"
              placeholder="Введите email..."
              class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Фильтр по стране</label>
            <input
              v-model="filters.developedCountries"
              type="text"
              placeholder="Введите страну..."
              class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>
        </div>

        <!-- Кнопка очистки фильтров -->
        <div v-if="activeFiltersCount > 0" class="mb-4 flex justify-end">
          <button
            @click="clearFilters"
            class="px-4 py-2 bg-red-100 text-red-700 rounded-md hover:bg-red-200 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors"
          >
            Очистить все фильтры ({{ activeFiltersCount }})
          </button>
        </div>

        <!-- Таблица -->
        <BaseTable 
          :rows="filteredData" 
          :headers="headers"
        >
          <!-- Слоты для кастомизации ячеек -->
          <template #email="{ value }">
            <a 
              :href="`mailto:${value}`" 
              class="text-blue-600 hover:text-blue-800 hover:underline"
            >
              {{ value }}
            </a>
          </template>

          <template #phone="{ value }">
            <PhoneFormatter :value="value" />
          </template>

          <template #image="{ value }">
            <img 
              :src="value" 
              alt="Avatar" 
              class="w-12 h-12 rounded-full object-cover border border-gray-200"
              @error="handleImageError"
            />
          </template>

          <template #marketCap="{ value }">
            <MarketCapCell :value="value" />
          </template>

          <template #developedCountries="{ value }">
            <CountryBadges :value="value" />
          </template>
        </BaseTable>
      </div>

      <!-- Статистика -->
      <div class="mt-6 grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="bg-white rounded-lg shadow p-4">
          <h3 class="text-lg font-semibold text-gray-700">Всего записей</h3>
          <p class="text-3xl font-bold text-blue-600">{{ data.length }}</p>
        </div>
        <div class="bg-white rounded-lg shadow p-4">
          <h3 class="text-lg font-semibold text-gray-700">Отображено</h3>
          <p class="text-3xl font-bold text-green-600">{{ filteredData.length }}</p>
        </div>
        <div class="bg-white rounded-lg shadow p-4">
          <h3 class="text-lg font-semibold text-gray-700">Активных фильтров</h3>
          <p class="text-3xl font-bold text-orange-600">{{ activeFiltersCount }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import BaseTable from './components/BaseTable.vue'
import MarketCapCell from './components/MarketCapCell.vue'
import CountryBadges from './components/CountryBadges.vue'
import PhoneFormatter from './components/PhoneFormatter.vue'

// Заголовки таблицы
const headers = ref({
  firstName: 'Имя',
  lastName: 'Фамилия',
  email: 'Email',
  phone: 'Телефон',
  image: 'Фото',
  marketCap: 'Рыночная капитализация',
  developedCountries: 'Развитые страны'
})

// Данные (пока пустые, загрузим из JSON)
const data = ref([])

// Фильтры
const filters = ref({
  firstName: '',
  lastName: '',
  email: '',
  developedCountries: ''
})

// Загрузка данных из JSON файла
const loadData = async () => {
  try {
    // Импортируем данные напрямую (если файл в src/data/)
    const response = await fetch('./src/data/data.json')
    if (!response.ok) throw new Error('Ошибка загрузки данных')
    data.value = await response.json()
  } catch (error) {
    console.error('Ошибка загрузки данных:', error)
    // Резервные данные на случай ошибки
    data.value = getFallbackData()
  }
}

// Резервные данные
const getFallbackData = () => {
  return [
    {
      "firstName": "Алексей",
      "lastName": "Смирнов",
      "email": "alexey.smirnov@example.com",
      "phone": "+7 912 345 67 89",
      "image": "https://picsum.photos/200?random=1",
      "marketCap": 1500000000,
      "developedCountries": ["Germany", "France", "Japan", "USA"]
    },
    {
      "firstName": "Мария",
      "lastName": "Иванова",
      "email": "maria.ivanova@example.com",
      "phone": "8 (912) 567-89-01",
      "image": "https://picsum.photos/200?random=2",
      "marketCap": 25000000,
      "developedCountries": ["United Kingdom", "Sweden", "Canada"]
    }
  ]
}

// Отфильтрованные данные
const filteredData = computed(() => {
  if (!hasActiveFilters.value) {
    return data.value
  }

  return data.value.filter(row => {
    return Object.entries(filters.value).every(([key, filterValue]) => {
      if (!filterValue || filterValue.trim() === '') return true
      
      const searchTerm = filterValue.toLowerCase().trim()
      const cellValue = row[key]
      
      if (!cellValue) return false
      
      // Для строк
      if (typeof cellValue === 'string') {
        return cellValue.toLowerCase().includes(searchTerm)
      }
      
      // Для чисел
      if (typeof cellValue === 'number') {
        return cellValue.toString().includes(searchTerm)
      }
      
      // Для массивов (стран)
      if (Array.isArray(cellValue)) {
        return cellValue.some(item => 
          item.toLowerCase().includes(searchTerm)
        )
      }
      
      return true
    })
  })
})

// Активные фильтры
const activeFiltersCount = computed(() => {
  return Object.values(filters.value).filter(v => v && v.trim()).length
})

const hasActiveFilters = computed(() => activeFiltersCount.value > 0)

// Очистка фильтров
const clearFilters = () => {
  filters.value = {
    firstName: '',
    lastName: '',
    email: '',
    developedCountries: ''
  }
}

// Обработчик ошибок изображений
const handleImageError = (event) => {
  event.target.src = 'https://via.placeholder.com/200?text=No+Image'
}

// Загружаем данные при монтировании
onMounted(() => {
  loadData()
})
</script>