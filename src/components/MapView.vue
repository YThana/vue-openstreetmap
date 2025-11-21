<script setup>
import { onMounted, onBeforeUnmount, ref, watch } from 'vue'
import * as L from 'leaflet'
import 'leaflet.markercluster'

const props = defineProps({
  markers: { type: Array, required: true },
  selectedMarkerId: { type: String, default: null },
  searchedPlace: { type: Object, default: null },
  userLocation: { type: Object, default: null },
})
const emit = defineEmits(['marker-clicked'])

const mapEl = ref(null)
let map
let cluster
let searchMarker = null
let userMarker = null
let userAccuracyCircle = null

function styleFor(marker, isSelected) {
  return {
    radius: isSelected ? 12 : 8,
    color: isSelected ? '#111' : '#333',
    weight: isSelected ? 3 : 1.5,
    fillColor: isSelected ? marker.selectedColor || marker.color : marker.color,
    fillOpacity: isSelected ? 0.9 : 0.7,
  }
}

function renderMarkers() {
  if (!map) return
  if (cluster) {
    cluster.clearLayers()
  } else {
    cluster = L.markerClusterGroup({
      showCoverageOnHover: false,
      spiderfyOnMaxZoom: true,
      maxClusterRadius: 60,
    }).addTo(map)
  }
  const items = props.markers || []
  items.forEach(m => {
    const isSelected = props.selectedMarkerId && props.selectedMarkerId === m.id
    const size = isSelected ? 20 : 14
    const border = isSelected ? 3 : 2
    const borderColor = isSelected ? '#111' : '#333'
    const fill = isSelected ? (m.selectedColor || m.color) : m.color
    const icon = L.divIcon({
      className: 'dot-icon',
      html: `<div style="width:${size}px;height:${size}px;border-radius:50%;background:${fill};border:${border}px solid ${borderColor};"></div>`,
      iconSize: [size + border*2, size + border*2],
      iconAnchor: [ (size + border*2)/2, (size + border*2)/2 ],
    })
    const mk = L.marker([m.lat, m.lng], { icon })
    mk.on('click', () => emit('marker-clicked', m))
    mk.bindTooltip(m.title || '', { sticky: true })
    cluster.addLayer(mk)
  })
  // Fit bounds if there are markers
  if (items.length > 0) {
    const bounds = cluster.getBounds()
    if (bounds && bounds.isValid && bounds.isValid()) {
      map.fitBounds(bounds.pad(0.2), { animate: false })
    } else {
      const b = L.latLngBounds(items.map(m => [m.lat, m.lng]))
      map.fitBounds(b.pad(0.2), { animate: false })
    }
  }
}

function handleSearchedPlace(place) {
  if (!map || !place) return

  // Remove previous search marker if exists
  if (searchMarker) {
    map.removeLayer(searchMarker)
    searchMarker = null
  }

  // Create a custom icon for the search result
  const icon = L.divIcon({
    className: 'search-marker-icon',
    html: `<div style="width:24px;height:24px;border-radius:50%;background:#3b82f6;border:3px solid #fff;box-shadow:0 2px 8px rgba(0,0,0,0.3);"></div>`,
    iconSize: [30, 30],
    iconAnchor: [15, 15],
  })

  // Add marker for searched location
  searchMarker = L.marker([place.lat, place.lng], { icon })
    .addTo(map)
    .bindPopup(`<b>${place.name}</b>`)
    .openPopup()

  // Pan and zoom to the location
  map.setView([place.lat, place.lng], 15, { animate: true })
}

function handleUserLocation(location) {
  if (!map || !location) return

  // Remove previous user marker and accuracy circle if exists
  if (userMarker) {
    map.removeLayer(userMarker)
    userMarker = null
  }
  if (userAccuracyCircle) {
    map.removeLayer(userAccuracyCircle)
    userAccuracyCircle = null
  }

  // Create a pulsing dot icon for user location
  const icon = L.divIcon({
    className: 'user-location-icon',
    html: `
      <div style="position:relative;width:40px;height:40px;">
        <div style="position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);width:16px;height:16px;border-radius:50%;background:#3b82f6;border:3px solid #fff;box-shadow:0 0 0 2px #3b82f6;z-index:2;"></div>
        <div style="position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);width:40px;height:40px;border-radius:50%;background:rgba(59,130,246,0.3);animation:pulse-location 2s ease-out infinite;"></div>
      </div>
    `,
    iconSize: [40, 40],
    iconAnchor: [20, 20],
  })

  // Add marker for user location
  userMarker = L.marker([location.lat, location.lng], { icon })
    .addTo(map)
    .bindPopup('<b>Your Location</b>')

  // Add accuracy circle if accuracy is available
  if (location.accuracy) {
    userAccuracyCircle = L.circle([location.lat, location.lng], {
      radius: location.accuracy,
      color: '#3b82f6',
      fillColor: '#3b82f6',
      fillOpacity: 0.1,
      weight: 1,
      opacity: 0.3
    }).addTo(map)
  }

  // Pan and zoom to user location
  map.setView([location.lat, location.lng], 15, { animate: true })
}

onMounted(() => {
  map = L.map(mapEl.value, { preferCanvas: true })
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors',
    maxZoom: 19,
  }).addTo(map)
  map.setView([40.73, -73.99], 12)
  renderMarkers()
})

onBeforeUnmount(() => {
  if (map) map.remove()
})

watch(() => props.markers, () => renderMarkers(), { deep: true })
watch(() => props.selectedMarkerId, () => renderMarkers())
watch(() => props.searchedPlace, (newPlace) => {
  if (newPlace) handleSearchedPlace(newPlace)
}, { deep: true })
watch(() => props.userLocation, (newLocation) => {
  if (newLocation) handleUserLocation(newLocation)
}, { deep: true })
</script>

<template>
  <div ref="mapEl" class="map"></div>
</template>

<style>
.map {
  flex: 1 1 auto;
}

@keyframes pulse-location {
  0% {
    transform: translate(-50%, -50%) scale(0.3);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 0;
  }
}
</style>
