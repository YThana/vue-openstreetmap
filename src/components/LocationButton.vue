<script setup>
import { ref } from 'vue'

const emit = defineEmits(['location-found', 'location-error'])

const isLoading = ref(false)
const error = ref(null)

async function getUserLocation() {
  if (!navigator.geolocation) {
    error.value = 'Geolocation is not supported by your browser'
    emit('location-error', error.value)
    return
  }

  isLoading.value = true
  error.value = null

  navigator.geolocation.getCurrentPosition(
    (position) => {
      isLoading.value = false
      emit('location-found', {
        lat: position.coords.latitude,
        lng: position.coords.longitude,
        accuracy: position.coords.accuracy
      })
    },
    (err) => {
      isLoading.value = false
      let errorMessage = 'Unable to retrieve your location'

      switch (err.code) {
        case err.PERMISSION_DENIED:
          errorMessage = 'Location permission denied'
          break
        case err.POSITION_UNAVAILABLE:
          errorMessage = 'Location information unavailable'
          break
        case err.TIMEOUT:
          errorMessage = 'Location request timed out'
          break
      }

      error.value = errorMessage
      emit('location-error', errorMessage)
    },
    {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0
    }
  )
}
</script>

<template>
  <button
    class="location-button"
    @click="getUserLocation"
    :disabled="isLoading"
    :title="error || 'Get my location'"
  >
    <span v-if="isLoading" class="icon">⌛</span>
    <span v-else-if="error" class="icon error">⚠️</span>
    <span v-else class="icon">📍</span>
  </button>
</template>

<style scoped>
.location-button {
  padding: 8px 12px;
  background: white;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 40px;
  min-height: 36px;
}

.location-button:hover:not(:disabled) {
  background-color: #f9fafb;
  border-color: #3b82f6;
}

.location-button:active:not(:disabled) {
  background-color: #f3f4f6;
}

.location-button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.icon {
  display: inline-block;
}

.icon.error {
  animation: shake 0.5s;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-4px); }
  75% { transform: translateX(4px); }
}
</style>
