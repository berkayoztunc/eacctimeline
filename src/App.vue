<script setup>
import { ref, computed, onMounted } from 'vue'

// Replace with your Google Sheet details
const SHEET_ID = '1jkHhRoxzrL04_c6_Ms9C4e9Vhip7V2BVL3keOUdj_Jk'
const API_KEY = 'AIzaSyB8hpVFnRYqeCEEQe9hdSoj5RjYXKmQZy0'
const RANGE = 'technological_advancements_unique_combination_final_corrected_expanded_retry!A1:Z'

const items = ref([])
const loading = ref(true)
const error = ref(null)
const scrollContainer = ref(null)

// Drag to scroll state
const isDragging = ref(false)
const startX = ref(0)
const scrollLeft = ref(0)

// Group and sort items by year within each vertical
const groupedItems = computed(() => {
  const groups = {}
  items.value.forEach(item => {
    if (!groups[item.vertical]) {
      groups[item.vertical] = []
    }
    groups[item.vertical].push(item)
  })

  Object.keys(groups).forEach(vertical => {
    groups[vertical].sort((a, b) => {
      return parseInt(a.start_year) - parseInt(b.start_year)
    })
  })

  return groups
})

const verticals = computed(() => {
  return Object.keys(groupedItems.value)
})

const fetchSheetData = async () => {
  try {
    const response = await fetch(
      `https://sheets.googleapis.com/v4/spreadsheets/${SHEET_ID}/values/${RANGE}?key=${API_KEY}`
    )
    const data = await response.json()

    const headers = data.values[0].map(header => header.toLowerCase().replace(/ /g, '_'))
    const rows = data.values.slice(1)

    items.value = rows.map(row => {
      const item = {}
      headers.forEach((header, index) => {
        item[header] = row[index]
      })
      return item
    })

    loading.value = false
  } catch (e) {
    error.value = 'Error fetching data from Google Sheets'
    loading.value = false
    console.error('Error:', e)
  }
}

// Drag to scroll handlers
const startDragging = (e) => {
  isDragging.value = true
  startX.value = e.pageX - scrollContainer.value.offsetLeft
  scrollLeft.value = scrollContainer.value.scrollLeft
}

const stopDragging = () => {
  isDragging.value = false
}

const move = (e) => {
  if (!isDragging.value) return
  e.preventDefault()
  const x = e.pageX - scrollContainer.value.offsetLeft
  const walk = x - startX.value
  scrollContainer.value.scrollLeft = scrollLeft.value - walk
}

onMounted(() => {
  fetchSheetData()
})
</script>

<template>
  <div class="min-h-screen bg-black">
    <!-- Loading State -->
    <div v-if="loading" class="text-center py-8">
      <h2 class="text-xl font-semibold text-cyan-400">Loading...</h2>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="text-red-500 text-center py-8">
      {{ error }}
    </div>

    <!-- Scrollable Container -->
    <div v-else class="relative">
      <!-- Sticky Header Row -->


      <!-- Scrollable Content -->
      <div class="flex gap-6 p-4 overflow-x-auto" ref="scrollContainer" @mousedown="startDragging" @mousemove="move"
        @mouseup="stopDragging" @mouseleave="stopDragging"
        :class="{ 'cursor-grabbing': isDragging, 'cursor-grab': !isDragging }">
        <!-- Column for each vertical -->
        <div v-for="vertical in verticals" :key="vertical">
                   <div class="flex h-16 w-80 bg-black border-b border-cyan-400/20 justify-center items-center">
            <h2 class="text-cyan-400 text-xl font-bold text-center">
              {{ vertical }}
            </h2>
          </div>

          <!-- Cards Stack -->
          <div class="space-y-4">
            <div v-for="item in groupedItems[vertical]" :key="item.milestone"
              class="bg-gray-900/95 border border-cyan-400 rounded-lg p-4 hover:bg-gray-800/95 transition-colors select-none">
              <div class="text-cyan-400 font-semibold mb-2">{{ item.milestone }}</div>
              <div class="space-y-2">
                <div>
                  <span class="text-cyan-400">Innovation:</span>
                  <span class="text-gray-300 ml-1">{{ item.innovation }}</span>
                </div>
                <div>
                  <span class="text-cyan-400">Milestone:</span>
                  <span class="text-gray-300 ml-1">{{ item.milestone }}</span>

                </div>
                <div>
                  <span class="text-cyan-400">Branch:</span>
                  <span class="text-gray-300 ml-1">{{ item.vertical }}</span>
                </div>
                <div>
                  <span class="text-cyan-400">Status:</span>
                  <span :class="{
                    'text-green-400': item.status === 'Complete',
                    'text-yellow-400': item.status === 'In Progress',
                    'text-red-400': item.status === 'Not Started'
                  }" class="ml-1">
                    {{ item.status }}
                  </span>
                </div>
                <div class="text-gray-400">
                  <span class="font-medium">Year:</span> {{ item.start_year }}
                  <span v-if="item.completion_year"> to {{ item.completion_year }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Hide scrollbar but keep functionality */
.hide-scrollbar {
  scrollbar-width: none;
  -ms-overflow-style: none;
}

.hide-scrollbar::-webkit-scrollbar {
  display: none;
}  

/* Smooth scrolling */
.overflow-x-auto {
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

/* Prevent text selection while dragging */
.select-none {
  user-select: none;
}
</style>