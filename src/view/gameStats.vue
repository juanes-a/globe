<template>
  <NavBar :isNavCollapsed="false" />
  <div class="jungle-container">
    <!-- Cabecera estilo jungla arcade -->
    <div class="jungle-header">
      <h1 class="jungle-title">🐾 Wild Arcade Tracker 🌿</h1>
      <div class="animal-health-bar">
        <div class="health-fill"></div>
        <div class="claw-marks"></div>
      </div>
    </div>

    <!-- Buscador estilo selvático -->
    <div class="jungle-search">
      <div class="vines-top">
        <div class="banana"></div>
        <div class="coconut"></div>
        <div class="mango"></div>
      </div>
      <input
        v-model="searchText"
        @input="applyFilter"
        type="text"
        placeholder=">> Track by game, territory or den..."
        class="jungle-input"
      />
      <div class="vines-bottom">
        <span class="animal-text">SCENT: FRESH</span>
        <span class="animal-text">PREY: {{ filteredPins.length }}</span>
      </div>
    </div>

    <!-- Lista de pins con estilo animalístico -->
    <ul class="den-list" v-if="filteredPins && filteredPins.length > 0">
      <li v-for="pin in filteredPins" :key="pin.id" class="den-item">
        <div class="paw-border">
          <div class="den-header">
            <span class="fur-pattern">PACK {{ Math.floor(Math.random() * 50) + 1 }}</span>
            <strong class="prey-name">🐾 {{ pin.game }}</strong>
          </div>
          
          <div class="den-content">
            <p class="animal-text">🌿 <span class="label">Mark:</span> {{ pin.note || 'No mark' }}</p>
            <p class="animal-text">🐺 <span class="label">Alpha:</span> {{ pin.username }}</p>
            <p class="animal-text">🗺️ <span class="label">Territory:</span> {{ pin.location?.country || 'Unknown' }}</p>
            <p class="animal-text">🏞️ <span class="label">Den:</span> {{ pin.location?.city || 'Unknown' }}</p>
            <p class="animal-text">📍 <span class="label">Trail:</span> 
              {{ pin.lat ? Number(pin.lat).toFixed(4) : 'N/A' }}, 
              {{ pin.lng ? Number(pin.lng).toFixed(4) : 'N/A' }}
            </p>
          </div>
          
          <div class="den-footer">
            <span class="animal-text">STRENGTH: ▮▮▮▮▮ 100%</span>
            <button class="paw-button">FOLLOW TRAIL</button>
          </div>
        </div>
      </li>
    </ul>

    <!-- Pantalla de hibernación cuando no hay resultados -->
    <div class="hibernation-screen" v-else>
      <div class="hibernation-content">
        <h2>HIBERNATION MODE</h2>
        <p class="animal-text">No tracks found in this territory</p>
        <button class="paw-button" @click="resetSearch">HUNT AGAIN?</button>
      </div>
    </div>

    <!-- Huellas animales decorativas -->
    <div class="paw-print top-left">🐾</div>
    <div class="paw-print top-right">🐾</div>
    <div class="paw-print bottom-left">🐾</div>
    <div class="paw-print bottom-right">🐾</div>
    
    <!-- Efecto de follaje -->
    <div class="jungle-foliage"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'
import NavBar from '../components/NavBar.vue'

const MAPBOX_TOKEN = 'pk.eyJ1IjoianVhbmVzNzgiLCJhIjoiY21id2xycnhrMTFvazJscTFidGFod2JwZiJ9.2d50Rw2voOOdpAg6GtoHAA'

const pins = ref<any[]>([])
const filteredPins = ref<any[]>([])
const searchText = ref('')

async function reverseGeocode(lat: number | undefined, lng: number | undefined) {
  if (!lat || !lng) return { country: null, city: null }
  
  try {
    const res = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${lng},${lat}.json?access_token=${MAPBOX_TOKEN}`)
    const features = res.data.features

    const country = features.find(f => f.place_type.includes('country'))?.text
    const city = features.find(f => f.place_type.includes('place') || f.place_type.includes('locality'))?.text

    return { country, city }
  } catch {
    return { country: null, city: null }
  }
}

onMounted(async () => {
  try {
    const response = await axios.get('http://localhost:8000/api/pins')
    const loadedPins = response.data

    for (const pin of loadedPins) {
      pin.location = await reverseGeocode(pin.lat, pin.lng)
    }

    pins.value = loadedPins
    filteredPins.value = loadedPins
  } catch (error) {
    console.error('❌ Error tracking prey:', error)
    filteredPins.value = []
  }
})

function applyFilter() {
  const q = searchText.value.trim().toLowerCase()
  if (!q) {
    filteredPins.value = pins.value
  } else {
    filteredPins.value = pins.value.filter(pin => {
      const country = pin.location?.country?.toLowerCase() || ''
      const city = pin.location?.city?.toLowerCase() || ''
      const game = pin.game?.toLowerCase() || ''
      
      return game.includes(q) || country.includes(q) || city.includes(q)
    })
  }
}

function resetSearch() {
  searchText.value = ''
  applyFilter()
}
</script>

<style scoped>
/* Estilo general jungla/animalístico */
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Rubik+Bubbles&display=swap');

* {
  box-sizing: border-box;
}

.jungle-container {
  position: relative;
  margin-top: 39px;
  padding: 30px;
  background-color: #0d1a0d;
  background-image: radial-gradient(#1e3a1e 1px, transparent 1px);
  background-size: 20px 20px;
  color: #e6e6e6;
  min-height: 100vh;
  font-family: 'Courier New', monospace;
  border: 8px solid #3a2c18;
  box-shadow: inset 0 0 30px #000;
  overflow: hidden;
}

.jungle-header {
  text-align: center;
  margin-bottom: 30px;
  position: relative;
}

.jungle-title {
  font-family: 'Rubik Bubbles', cursive;
  color: #f0a830;
  text-shadow: 3px 3px 0 #8b4513, 6px 6px 0 #5a3921;
  margin-bottom: 20px;
  font-size: 2.5rem;
  letter-spacing: 2px;
}

.animal-health-bar {
  height: 16px;
  background: #3a2c18;
  border-radius: 8px;
  margin: 0 auto;
  max-width: 500px;
  border: 2px solid #5a3921;
  box-shadow: 0 0 10px #8b4513;
  position: relative;
  overflow: hidden;
}

.health-fill {
  height: 100%;
  width: 100%;
  background: linear-gradient(90deg, #ff4500, #ff8c00, #32cd32);
  border-radius: 6px;
}

.claw-marks {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("data:image/svg+xml,%3Csvg width='20' height='20' viewBox='0 0 20 20' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M5 5h2v2H5zM8 8h2v2H8zM11 11h2v2h-2zM14 14h2v2h-2z' fill='%23000' opacity='0.2'/%3E%3C/svg%3E");
}

/* Estilo para el buscador jungla */
.jungle-search {
  background: #1e3a1e;
  border: 4px solid #5a3921;
  border-radius: 8px;
  padding: 10px;
  max-width: 600px;
  margin: 0 auto 30px;
  box-shadow: 0 0 15px rgba(139, 69, 19, 0.5);
  position: relative;
}

.vines-top {
  display: flex;
  gap: 15px;
  margin-bottom: 10px;
  justify-content: center;
}

.banana, .coconut, .mango {
  width: 20px;
  height: 20px;
  border-radius: 50%;
}

.banana {
  background: #ffd700;
  box-shadow: 0 0 5px #ffd700;
}

.coconut {
  background: #8b4513;
  box-shadow: 0 0 5px #8b4513;
}

.mango {
  background: #ff4500;
  box-shadow: 0 0 5px #ff4500;
}

.jungle-input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  background: #0d1a0d;
  color: #f0a830;
  border: 2px solid #5a3921;
  font-family: 'Courier New', monospace;
  margin-bottom: 10px;
  border-radius: 4px;
}

.jungle-input::placeholder {
  color: #5a3921;
  font-style: italic;
}

.jungle-input:focus {
  outline: none;
  border-color: #f0a830;
  box-shadow: 0 0 10px #f0a830;
}

.vines-bottom {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #f0a830;
  padding: 0 5px;
}

/* Estilo para la lista de guaridas */
.den-list {
  list-style: none;
  padding: 0;
  max-width: 800px;
  margin: 0 auto;
}

.den-item {
  margin-bottom: 25px;
  transition: transform 0.3s;
}

.den-item:hover {
  transform: translateY(-5px);
}

.paw-border {
  border: 4px solid #5a3921;
  border-image: repeating-linear-gradient(45deg, #8b4513, #f0a830, #8b4513, #f0a830) 10;
  padding: 15px;
  background: rgba(13, 26, 13, 0.8);
  position: relative;
  border-radius: 8px;
}

.den-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  border-bottom: 2px dashed #5a3921;
  padding-bottom: 10px;
}

.fur-pattern {
  background: #8b4513;
  color: #f0a830;
  padding: 3px 8px;
  font-size: 12px;
  font-family: 'Press Start 2P', cursive;
  border-radius: 4px;
}

.prey-name {
  color: #f0a830;
  font-size: 1.3rem;
  text-shadow: 2px 2px 0 #8b4513;
  font-family: 'Rubik Bubbles', cursive;
}

.den-content {
  margin-bottom: 15px;
}

.animal-text {
  margin: 8px 0;
  color: #e6e6e6;
  font-size: 14px;
}

.label {
  color: #32cd32;
  font-weight: bold;
}

.den-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 2px dashed #5a3921;
  padding-top: 10px;
}

/* Estilo para botones de pata */
.paw-button {
  background: #5a3921;
  color: #f0a830;
  border: none;
  padding: 8px 15px;
  font-family: 'Press Start 2P', cursive;
  font-size: 12px;
  cursor: pointer;
  position: relative;
  text-transform: uppercase;
  border-radius: 20px;
  transition: all 0.3s;
}

.paw-button:hover {
  background: #8b4513;
  color: #ffd700;
  transform: scale(1.05);
}

.paw-button:active {
  transform: scale(0.95);
}

/* Pantalla de hibernación */
.hibernation-screen {
  background: rgba(13, 26, 13, 0.9);
  border: 4px solid #8b4513;
  padding: 30px;
  text-align: center;
  max-width: 500px;
  margin: 50px auto;
  border-radius: 10px;
}

.hibernation-screen h2 {
  color: #f0a830;
  font-family: 'Rubik Bubbles', cursive;
  font-size: 2.5rem;
  margin-bottom: 20px;
  text-shadow: 3px 3px 0 #8b4513;
}

/* Huellas decorativas */
.paw-print {
  position: absolute;
  font-size: 24px;
  opacity: 0.3;
  z-index: 1;
}

.top-left {
  top: 20px;
  left: 20px;
}

.top-right {
  top: 20px;
  right: 20px;
}

.bottom-left {
  bottom: 20px;
  left: 20px;
}

.bottom-right {
  bottom: 20px;
  right: 20px;
}

/* Follaje decorativo */
.jungle-foliage {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("data:image/svg+xml,%3Csvg width='100' height='100' viewBox='0 0 100 100' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M10 10 Q 20 0, 30 10 T 50 10 T 70 10 T 90 10' stroke='%231e3a1e' fill='none' stroke-width='2'/%3E%3C/svg%3E");
  background-size: 200px 200px;
  pointer-events: none;
  z-index: 0;
  opacity: 0.1;
}

/* Animaciones */
@keyframes growl {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

.den-item:hover {
  animation: growl 0.5s infinite;
}

/* Responsive */
@media (max-width: 768px) {
  .jungle-title {
    font-size: 1.8rem;
  }
  
  .jungle-container {
    padding: 20px;
  }
  
  .prey-name {
    font-size: 1.1rem;
  }
}
</style>