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
        <!-- Таблица с кастомными заголовками и фильтрами -->
        <BaseTable 
          :rows="filteredData" 
          :headers="headers"
          :active-filters="filters"
          @filter-change="handleFilterChange"
          @clear-filter="clearColumnFilter"
        >
          <!-- Слоты для кастомных заголовков с фильтрами -->
          <template #header-firstName="{ key, text }">
            <div class="space-y-2">
              <span class="font-medium">{{ text }}</span>
              <InputFilter
                v-model="filterModels.firstName"
                :column="key"
                placeholder="Поиск по имени..."
                @filter="handleFilter"
              />
            </div>
          </template>

          <template #header-lastName="{ key, text }">
            <div class="space-y-2">
              <span class="font-medium">{{ text }}</span>
              <InputFilter
                v-model="filterModels.lastName"
                :column="key"
                placeholder="Поиск по фамилии..."
                @filter="handleFilter"
              />
            </div>
          </template>

          <template #header-email="{ key, text }">
            <div class="space-y-2">
              <span class="font-medium">{{ text }}</span>
              <InputFilter
                v-model="filterModels.email"
                :column="key"
                placeholder="Поиск по email..."
                @filter="handleFilter"
              />
            </div>
          </template>

          <template #header-phone="{ key, text }">
            <div class="space-y-2">
              <span class="font-medium">{{ text }}</span>
              <InputFilter
                v-model="filterModels.phone"
                :column="key"
                placeholder="Поиск по телефону..."
                @filter="handleFilter"
              />
            </div>
          </template>

          <template #header-developedCountries="{ key, text }">
            <div class="space-y-2">
              <span class="font-medium">{{ text }}</span>
              <InputFilter
                v-model="filterModels.developedCountries"
                :column="key"
                placeholder="Поиск по стране..."
                @filter="handleFilter"
              />
            </div>
          </template>

          <!-- Слоты для кастомизации ячеек -->
          <template #email="{ value }">
            <a 
              :href="`mailto:${value}`" 
              class="text-blue-600 hover:text-blue-800 hover:underline transition-colors duration-150"
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
import { ref, computed, onMounted, watch, shallowRef } from 'vue'
import debounce from 'lodash.debounce'
import BaseTable from './components/BaseTable.vue'
import MarketCapCell from './components/MarketCapCell.vue'
import CountryBadges from './components/CountryBadges.vue'
import PhoneFormatter from './components/PhoneFormatter.vue'
import InputFilter from './components/InputFilter.vue'

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

// Исходные данные - используем shallowRef для оптимизации
const data = shallowRef([])

// Модели для v-model в InputFilter (сохраняют значения)
const filterModels = ref({
  firstName: '',
  lastName: '',
  email: '',
  phone: '',
  developedCountries: ''
})

// Активные фильтры (только непустые значения) - тоже shallowRef
const filters = shallowRef({})

// Отфильтрованные данные
const filteredData = shallowRef([])

// Дебаунсинг фильтрации (300мс задержка)
const applyFiltersDebounced = debounce(() => {
  if (Object.keys(filters.value).length === 0) {
    filteredData.value = data.value
    return
  }

  filteredData.value = data.value.filter(row => {
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
      
      // Для массивов
      if (Array.isArray(cellValue)) {
        return cellValue.some(item => 
          item.toLowerCase().includes(searchTerm)
        )
      }
      
      return true
    })
  })
}, 300)

// Следим за изменениями фильтров и данных
watch(() => filters.value, () => {
  applyFiltersDebounced()
}, { deep: true })

watch(() => data.value, () => {
  applyFiltersDebounced()
}, { immediate: true })

// Вычисляемые свойства
const activeFiltersCount = computed(() => {
  return Object.keys(filters.value).length
})

// Загрузка данных
const loadData = async () => {
  try {
    // Если данные уже есть, не загружаем снова
    if (data.value.length > 0) return
    
    // Используем статический импорт для стабильности
    // import usersData from './data/data.json'
    // data.value = usersData
    
    const response = await fetch('./src/data/data.json')
    if (!response.ok) throw new Error('Ошибка загрузки данных')
    const loadedData = await response.json()
    
    // Одной операцией назначаем все данные
    data.value = loadedData
    
  } catch (error) {
    console.error('Ошибка загрузки данных:', error)
    data.value = getFallbackData()
  }
}

// Обработчики событий
const handleFilter = debounce(({ key, value }) => {
  if (value && value.trim() !== '') {
    // Используем spread оператор для реактивности
    filters.value = { ...filters.value, [key]: value.trim() }
  } else {
    // Удаляем фильтр если значение пустое
    const newFilters = { ...filters.value }
    delete newFilters[key]
    filters.value = newFilters
  }
}, 150) // Меньшая задержка для немедленной реакции

const handleFilterChange = (newFilters) => {
  filters.value = newFilters
  // Синхронизируем модели
  Object.keys(filterModels.value).forEach(key => {
    if (!newFilters[key]) {
      filterModels.value[key] = ''
    }
  })
}

const clearColumnFilter = (key) => {
  const newFilters = { ...filters.value }
  delete newFilters[key]
  filters.value = newFilters
  filterModels.value[key] = ''
}

const handleImageError = (event) => {
  event.target.src = 'https://via.placeholder.com/200?text=No+Image'
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

// Загружаем данные при монтировании
onMounted(() => {
  loadData()
})

// Очищаем таймеры при размонтировании
import { onUnmounted } from 'vue'
onUnmounted(() => {
  applyFiltersDebounced.cancel()
  handleFilter.cancel()
})
</script>