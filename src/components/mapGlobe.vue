<template>
  <div>
    <div ref="mapContainer" class="map-container"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import 'mapbox-gl/dist/mapbox-gl.css'
import mapboxgl from 'mapbox-gl'

// ✅ Asegúrate de tener un token válido
mapboxgl.accessToken = 'pk.eyJ1IjoianVhbmVnYW1lciIsImEiOiJjbWJzcDk0a3EwcDFuMnJweGFjZWxreXg5In0.sO83DBip88pAvfRHa1_VKg'

const mapContainer = ref<HTMLElement | null>(null)

onMounted(() => {
  if (!mapContainer.value) {
    console.error('❌ mapContainer no está disponible')
    return
  }

  console.log('✅ mapContainer encontrado:', mapContainer.value)

  const map = new mapboxgl.Map({
    container: mapContainer.value,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 2,
    projection: 'globe'
  })

  map.on('style.load', () => {
    console.log('✅ Estilo cargado correctamente')
    map.setFog({
      color: 'rgb(186, 210, 235)',
      'high-color': 'rgb(36, 92, 223)',
      'horizon-blend': 0.02,
      'space-color': 'rgb(11, 11, 25)',
      'star-intensity': 0.6
    })
  })
})
</script>

<style scoped>
.map-container {
  width: 100vw;
  height: 100vh;
}
</style>
