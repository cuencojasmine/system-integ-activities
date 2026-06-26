<template>
  <v-container fluid class="pa-4">
    <v-row justify="center">
      <v-col cols="12" md="10" lg="8">
        <div class="text h-5 font-weight-bold">Interactive Map</div>
        <v-card rounded="lg" elevation="3">
          <div id="map" style="height: 500px; width: 100%; border-radius: inherit;">
          </div>
        </v-card>
        </v-col>
        </v-row>
        </v-container>
</template>

<script lang="ts" setup>
//@ts-nocheck
onMounted(async () => {
  if (import.meta.client) {
    const L = await import('leaflet');
    const map = L.map('map').setView([15.041451, 120.683178], 13);
    
    var marker = L.marker([15.041451, 120.683178]).addTo(map);

    marker.bindPopup("St. Nicolas College").openPopup();

    function onMapClick(e) {
      popup
        .setLatLng(e.latlng)
        .setContent("You clicked the map at " + e.latlng.toString())
        .openOn(map);
    }

    map.on('click', onMapClick);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);
  }
});
</script>

<style>
#map {
  min-height: 500px;
  border-radius: 8px;
}
</style>