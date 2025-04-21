<template>
  <div class="app">
    <h1>🌦 Weather Monitoring and Forecasting</h1>

    <!-- Select uchun stil -->
    <div class="select-wrapper">
      <select v-model="city" @change="fetchData" class="custom-select">
        <option v-for="c in cities" :key="c" :value="c">
          {{ c }}
        </option>
      </select>
      <ChevronDownIcon class="icon arrow-icon" />
    </div>

    <h2>Current Weather in {{ city }}</h2>

    <!-- Fade animatsiyasi bilan kartochkalar -->
    <transition name="fade" mode="out-in">
      <div v-if="weatherData" key="cards" class="cards-container">
        <div class="cards-list">
          <div
            v-for="(temp, idx) in weatherData.temperatures"
            :key="weatherData.timestamps[idx]"
            class="weather-card"
          >
            <ClockIcon class="icon card-icon" />
            <p class="timestamp">{{ weatherData.timestamps[idx] }}</p>
            <SunIcon class="icon card-icon" />
            <p class="temperature">{{ temp.toFixed(2) }}°C</p>
          </div>
        </div>
      </div>
    </transition>

    <h2>Forecast</h2>
    <p v-if="forecast">🌤 Predicted Temperature: {{ forecast.toFixed(2) }}°C</p>
  </div>
</template>

<script>
import { ClockIcon, ChevronDownIcon, SunIcon } from '@heroicons/vue/outline';

export default {
  components: { ClockIcon, ChevronDownIcon, SunIcon },
  data() {
    return {
      city: 'Tashkent',
      cities: ['Tashkent', 'Andijan', 'Bukhara'],
      weatherData: null,
      forecast: null,
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
      try {
        this.weatherData = null; // Fade-out boshlanishi uchun

        // API chaqiruvlari
        const weatherPromise = fetch(`http://192.168.1.40:8080/api/weather?city=${this.city}`).then(r => r.json());
        const forecastPromise = fetch(`http://192.168.1.40:8080/api/forecast?city=${this.city}`).then(r => r.json());

        const [weatherJson, forecastJson] = await Promise.all([weatherPromise, forecastPromise]);

        // 300ms kechikish bilan yangi ma'lumotlarni o'rnatish
        setTimeout(() => {
          this.weatherData = weatherJson;
          this.forecast = forecastJson.temperature;
        }, 300);
      } catch (e) {
        console.error(e);
      }
    },
  },
};
</script>

<style scoped>
.app {
  font-family: Arial, sans-serif;
  padding: 200px;
  text-align: center;
  background: url('assets/bg.jpg');
  background-size: cover;
  background-repeat: no-repeat;
  width: 100vw;
  height: 100vh;
}

/* Select uchun stil */
.select-wrapper {
  position: relative;
  display: inline-block;
  margin-bottom: 20px;
}
.custom-select {
  appearance: none;
  padding: 8px 32px 8px 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 1rem;
}
.arrow-icon {
  position: absolute;
  right: 10px;
  top: 50%;
  width: 20px;
  height: 20px;
  transform: translateY(-50%);
  pointer-events: none;
}

/* Kartochkalar uchun stil */
.cards-container {
  margin: 20px 0;
}
.cards-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 16px;
}
.weather-card {
  background: rgba(255, 255, 255, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.5);
  backdrop-filter: blur(8px);
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}
.card-icon {
  width: 24px;
  height: 24px;
  margin-bottom: 8px;
}
.timestamp {
  font-size: 0.85rem;
  color: #555;
  margin-bottom: 8px;
}
.temperature {
  font-size: 1.25rem;
  font-weight: bold;
}

/* Fade animatsiyasi */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
