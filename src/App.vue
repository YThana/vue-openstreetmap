<script setup>
import { ref, reactive, computed, watch } from 'vue'
import MapView from './components/MapView.vue'
import SourceSelector from './components/SourceSelector.vue'
import Sidebar from './components/Sidebar.vue'
import SearchBox from './components/SearchBox.vue'
import LocationButton from './components/LocationButton.vue'

const sources = [
  { id: 'restaurants', name: 'Restaurants', url: '/data/restaurants.json', color: '#e74c3c', selectedColor: '#c0392b' },
  { id: 'parks', name: 'Parks', url: '/data/parks.json', color: '#2ecc71', selectedColor: '#27ae60' },
]

const selectedSourceIds = ref([])
const sourceData = reactive({}) // { [sourceId]: array of items }
const selectedMarkerId = ref(null)
const searchedPlace = ref(null)
const userLocation = ref(null)

const markers = computed(() => {
  const list = []
  for (const src of sources) {
    if (!selectedSourceIds.value.includes(src.id)) continue
    const items = sourceData[src.id] || []
    for (const it of items) {
      list.push({
        id: `${src.id}:${it.id}`,
        lat: it.lat,
        lng: it.lng,
        title: it.name,
        description: it.description || it.address || '',
        sourceId: src.id,
        color: src.color,
        selectedColor: src.selectedColor,
        raw: it,
      })
    }
  }
  return list
})

async function ensureLoaded(sourceId) {
  if (sourceData[sourceId]) return
  const src = sources.find(s => s.id === sourceId)
  if (!src) return
  const res = await fetch(src.url)
  if (!res.ok) return
  sourceData[sourceId] = await res.json()
}

watch(selectedSourceIds, async (newVal, oldVal) => {
  // Load data for any newly-selected sources
  const newlySelected = newVal.filter(id => !(oldVal || []).includes(id))
  await Promise.all(newlySelected.map(id => ensureLoaded(id)))
  // Clear selection if selected marker is from an unselected source
  if (selectedMarkerId.value) {
    const [srcId] = selectedMarkerId.value.split(':')
    if (!newVal.includes(srcId)) selectedMarkerId.value = null
  }
}, { immediate: true })

const selectedItem = computed(() => {
  if (!selectedMarkerId.value) return null
  const m = markers.value.find(m => m.id === selectedMarkerId.value)
  return m || null
})

function handleMarkerClicked(marker) {
  selectedMarkerId.value = marker?.id || null
}

function clearSelection() {
  selectedMarkerId.value = null
}

function handlePlaceSelected(place) {
  searchedPlace.value = place
}

function handleLocationFound(location) {
  userLocation.value = location
}

function handleLocationError(error) {
  console.error('Location error:', error)
}
</script>

<template>
  <div class="app">
    <header class="toolbar">
      <SearchBox @place-selected="handlePlaceSelected" />
      <LocationButton @location-found="handleLocationFound" @location-error="handleLocationError" />
      <SourceSelector :sources="sources" v-model:selected="selectedSourceIds" />
    </header>
    <main class="content">
      <MapView :markers="markers" :selectedMarkerId="selectedMarkerId" :searchedPlace="searchedPlace" :userLocation="userLocation" @marker-clicked="handleMarkerClicked" />
      <!-- <Sidebar :item="selectedItem" @close="clearSelection" /> -->
    </main>
  </div>
</template>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}
.toolbar {
  padding: 8px 12px;
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  gap: 12px;
  align-items: center;
}
.content {
  display: flex;
  flex: 1 1 auto;
  min-height: 0;
}
</style>
