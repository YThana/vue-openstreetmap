<script setup>
import { ref, watch, nextTick } from 'vue'

const emit = defineEmits(['place-selected'])

const searchQuery = ref('')
const searchResults = ref([])
const isSearching = ref(false)
const showResults = ref(false)
const highlightedIndex = ref(-1)
const inputRef = ref(null)
const resultsRef = ref(null)
let searchTimeout = null

async function searchPlace(query) {
  if (!query || query.length < 3) {
    searchResults.value = []
    showResults.value = false
    return
  }

  isSearching.value = true

  try {
    const response = await fetch(
      `https://nominatim.openstreetmap.org/search?` +
      `format=json&q=${encodeURIComponent(query)}&limit=5&addressdetails=1`
    )

    if (response.ok) {
      const data = await response.json()
      searchResults.value = data
      showResults.value = data.length > 0
      highlightedIndex.value = -1
    }
  } catch (error) {
    console.error('Search error:', error)
    searchResults.value = []
  } finally {
    isSearching.value = false
  }
}

watch(searchQuery, (newQuery) => {
  clearTimeout(searchTimeout)
  if (newQuery.length < 3) {
    searchResults.value = []
    showResults.value = false
    return
  }
  searchTimeout = setTimeout(() => {
    searchPlace(newQuery)
  }, 300)
})

function selectPlace(place) {
  emit('place-selected', {
    lat: parseFloat(place.lat),
    lng: parseFloat(place.lon),
    name: place.display_name,
    boundingBox: place.boundingbox
  })
  searchQuery.value = place.display_name
  showResults.value = false
  highlightedIndex.value = -1
  // Blur the input after selection
  if (inputRef.value) {
    inputRef.value.blur()
  }
}

function handleBlur() {
  setTimeout(() => {
    showResults.value = false
    highlightedIndex.value = -1
  }, 200)
}

function handleFocus() {
  if (searchResults.value.length > 0) {
    showResults.value = true
  }
}

function scrollToHighlighted() {
  nextTick(() => {
    if (!resultsRef.value) return
    const highlightedElement = resultsRef.value.querySelector('.search-result-item.highlighted')
    if (highlightedElement) {
      highlightedElement.scrollIntoView({ block: 'nearest', behavior: 'smooth' })
    }
  })
}

function handleKeydown(event) {
  if (!showResults.value || searchResults.value.length === 0) return

  switch (event.key) {
    case 'ArrowDown':
      event.preventDefault()
      highlightedIndex.value = (highlightedIndex.value + 1) % searchResults.value.length
      scrollToHighlighted()
      break
    case 'ArrowUp':
      event.preventDefault()
      highlightedIndex.value = highlightedIndex.value <= 0
        ? searchResults.value.length - 1
        : highlightedIndex.value - 1
      scrollToHighlighted()
      break
    case 'Enter':
      event.preventDefault()
      const indexToSelect = highlightedIndex.value >= 0 ? highlightedIndex.value : 0
      if (searchResults.value[indexToSelect]) {
        selectPlace(searchResults.value[indexToSelect])
      }
      break
    case 'Escape':
      showResults.value = false
      highlightedIndex.value = -1
      break
  }
}
</script>

<template>
  <div class="search-box">
    <div class="search-input-wrapper">
      <input
        ref="inputRef"
        v-model="searchQuery"
        type="text"
        placeholder="Search for a place..."
        class="search-input"
        @focus="handleFocus"
        @blur="handleBlur"
        @keydown="handleKeydown"
      />
      <span v-if="isSearching" class="search-spinner">🔍</span>
    </div>

    <div v-if="showResults" ref="resultsRef" class="search-results">
      <div
        v-for="(result, index) in searchResults"
        :key="result.place_id"
        :class="['search-result-item', { highlighted: index === highlightedIndex }]"
        @click="selectPlace(result)"
        @mouseenter="highlightedIndex = index"
      >
        <div class="result-name">{{ result.display_name }}</div>
        <div class="result-type">{{ result.type }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.search-box {
  position: relative;
  width: 100%;
  max-width: 400px;
}

.search-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.search-input {
  width: 100%;
  padding: 8px 32px 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  outline: none;
  transition: border-color 0.2s;
}

.search-input:focus {
  border-color: #3b82f6;
}

.search-spinner {
  position: absolute;
  right: 10px;
  font-size: 16px;
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.search-results {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  margin-top: 4px;
  background: white;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  max-height: 300px;
  overflow-y: auto;
  z-index: 1000;
}

.search-result-item {
  padding: 10px 12px;
  cursor: pointer;
  border-bottom: 1px solid #f3f4f6;
  transition: background-color 0.15s;
}

.search-result-item:last-child {
  border-bottom: none;
}

.search-result-item:hover,
.search-result-item.highlighted {
  background-color: #f0f9ff;
}

.search-result-item.highlighted {
  border-left: 3px solid #3b82f6;
  padding-left: 9px;
}

.result-name {
  font-size: 14px;
  color: #111827;
  margin-bottom: 2px;
}

.result-type {
  font-size: 12px;
  color: #6b7280;
  text-transform: capitalize;
}
</style>
