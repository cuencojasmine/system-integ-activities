<!-- pages/maps.vue -->
<template>
  <div class="maps-bg" style="min-height: 100vh; padding: 48px 16px;">
    <v-container style="max-width: 860px;" class="pa-0">

      <!-- Header Card -->
      <v-card class="maps-card mb-4" elevation="0" rounded="xl">
        <div class="card-header d-flex flex-column align-center justify-center">
          <v-icon size="44" color="white" class="mb-3">mdi-map-outline</v-icon>
          <h1 class="header-title">Interactive Map</h1>
          <p class="header-sub">Click anywhere on the map to get coordinates</p>
        </div>

        <v-card-text class="pa-0">
          <!-- Map Container -->
          <div class="map-wrapper">
            <div id="map" class="map-el" />

            <!-- Scanning overlay while map loads -->
            <div v-if="!mapReady" class="map-loading">
              <v-progress-circular indeterminate color="#2B5748" size="40" width="3" />
              <p class="loading-text mt-3">Loading map...</p>
            </div>
          </div>
        </v-card-text>

        <!-- Status bar -->
        <div class="status-bar d-flex align-center gap-2 px-5 py-3">
          <v-icon size="14" :color="clickedCoords ? '#2B5748' : '#9e9e9e'">mdi-circle</v-icon>
          <span class="status-text">
            {{ clickedCoords ?? 'Map loaded — click to inspect a location' }}
          </span>
        </div>
      </v-card>

      <!-- Coords Detail Card (shows after click) -->
      <v-card v-if="parsedCoords" class="maps-card" elevation="0" rounded="xl">
        <v-card-text class="pa-6">
          <p class="section-label mb-4">
            <v-icon size="18" class="mr-1" style="color:#2B5748">mdi-crosshairs-gps</v-icon>
            Last Clicked Location
          </p>
          <div class="coords-row mb-2">
            <span class="coord-label">Latitude</span>
            <span class="coord-value">{{ parsedCoords.lat }}</span>
          </div>
          <div class="coords-row">
            <span class="coord-label">Longitude</span>
            <span class="coord-value">{{ parsedCoords.lng }}</span>
          </div>
        </v-card-text>
      </v-card>

    </v-container>
  </div>
</template>

<script lang="ts" setup>
// @ts-nocheck
import { ref, onMounted, computed } from 'vue'

const clickedCoords = ref<string | null>(null)
const mapReady = ref(false)

const parsedCoords = computed(() => {
  if (!clickedCoords.value) return null
  const match = clickedCoords.value.match(/LatLng\(([-\d.]+),\s*([-\d.]+)\)/)
  if (!match) return null
  return { lat: parseFloat(match[1]).toFixed(6), lng: parseFloat(match[2]).toFixed(6) }
})

onMounted(async () => {
  if (import.meta.client) {
    const L = await import('leaflet')
    await import('leaflet/dist/leaflet.css')

    const map = L.map('map').setView([15.041451, 120.683178], 13);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors',
    }).addTo(map)

    // Custom colored marker using divIcon
    const markerIcon = L.divIcon({
      className: '',
      html: `<div style="
        width: 18px; height: 18px;
        background: #2B5748;
        border: 3px solid #ffffff;
        border-radius: 50%;
        box-shadow: 0 2px 8px rgba(43,87,72,0.5);
      "></div>`,
      iconSize: [18, 18],
      iconAnchor: [9, 9],
    })

    L.marker([15.041451, 120.683178], { icon: markerIcon }).addTo(map)

    const popup = L.popup({
      className: 'custom-popup',
    })

    map.on('click', (e) => {
      const coords = e.latlng.toString()
      clickedCoords.value = coords

      // Add a click marker
      const clickIcon = L.divIcon({
        className: '',
        html: `<div style="
          width: 14px; height: 14px;
          background: #0F2D40;
          border: 3px solid #ffffff;
          border-radius: 50%;
          box-shadow: 0 2px 6px rgba(15,45,64,0.5);
        "></div>`,
        iconSize: [14, 14],
        iconAnchor: [7, 7],
      })

      popup
        .setLatLng(e.latlng)
        .setContent(
          `<div style="font-family:inherit;font-size:0.8rem;color:#0F2D40;font-weight:600;">
            📍 ${e.latlng.lat.toFixed(5)}, ${e.latlng.lng.toFixed(5)}
          </div>`
        )
        .openOn(map)
    })

    mapReady.value = true
  }
})
</script>

<style scoped>
.maps-bg {
  background: #F0FDF8;
}

.maps-card {
  background: #ffffff;
  border: 1px solid #c8e6d8;
  overflow: hidden;
}

.card-header {
  height: 190px;
  background: linear-gradient(135deg, #0F2D40 0%, #2B5748 100%);
  padding: 0 32px;
}

.header-title {
  font-size: 1.5rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0;
  letter-spacing: -0.01em;
}

.header-sub {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.72);
  margin: 4px 0 0;
}

.map-wrapper {
  position: relative;
  width: 100%;
  height: 460px;
}

.map-el {
  width: 100%;
  height: 100%;
}

.map-loading {
  position: absolute;
  inset: 0;
  background: #F0FDF8;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 10;
}

.loading-text {
  font-size: 0.875rem;
  color: #2B5748;
  font-weight: 500;
}

.status-bar {
  border-top: 1px solid #c8e6d8;
  background: #F0FDF8;
}

.status-text {
  font-size: 0.8rem;
  color: #6b7280;
  font-weight: 500;
}

.section-label {
  font-size: 0.8rem;
  font-weight: 700;
  color: #0F2D40;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  display: flex;
  align-items: center;
}

.coords-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 14px;
  background: #F0FDF8;
  border-radius: 10px;
  margin-bottom: 8px;
}

.coord-label {
  font-size: 0.8rem;
  color: #6b7280;
  font-weight: 500;
}

.coord-value {
  font-size: 0.9rem;
  font-weight: 700;
  color: #0F2D40;
  font-variant-numeric: tabular-nums;
}
</style>

<style>
/* Global: style the Leaflet popup to match palette */
.leaflet-popup-content-wrapper {
  border-radius: 10px !important; 
  border: 1px solid #c8e6d8 !important;
  box-shadow: 0 4px 16px rgba(43, 87, 72, 0.15) !important;
  padding: 4px 8px !important;
}

.leaflet-popup-tip {
  background: #ffffff !important;
}

.leaflet-popup-content {
  margin: 8px 10px !important;
}
</style>