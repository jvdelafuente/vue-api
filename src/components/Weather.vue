<template>
  <div class="bodyDiv">
    <video autoplay muted loop playsinline class="background-video" :src="videoSrc"></video>

    <!-- El input y el botón deben estar fuera del bloque "v-if" -->
    <div class="contentDiv">
      <button @click="getWeather">Get Weather</button>
      <input v-model="city" @keyup.enter="getWeather" placeholder="Enter city name" />

      <!-- Mostrar errores, si los hay -->
      <div class="error" v-if="error">{{ error }}</div>

      <!-- Mostrar datos del clima solo si están disponibles -->
      <div v-if="weather">
        <h2>{{ weather.name }}</h2>
        <p> <span>Temperature:</span>  {{ weather.main.temp }}°C</p>
        <p> <span>Feels Like:</span>  {{ weather.main.feels_like }}°C</p>
        <p> <span>Min Temperature:</span>  {{ weather.main.temp_min }}°C</p>
        <p> <span>Max Temperature:</span>  {{ weather.main.temp_max }}°C</p>
        <p> <span>Humidity:</span>  {{ weather.main.humidity }}%</p>
        <p> <span>Weather:</span>  {{ weather.weather[0].description }}</p>
        <p> <span>Wind Speed:</span>  {{ weather.wind.speed }} m/s</p>
        <p> <span>Wind Direction:</span>  {{ weather.wind.deg }}°</p>
        <p> <span>Cloudiness:</span>  {{ weather.clouds.all }}%</p>
      </div>
    </div>
  </div>
</template>

<script>
import sunVideo from '../assets/sun.mp4';
import cloudsVideo from '../assets/clouds.mp4';
import rainVideo from '../assets/raindrops.mp4';
import snowVideo from '../assets/snowfall.mp4';
import windVideo from '../assets/wind.mp4';

export default {
  data() {
    return {
      city: '',
      weather: null,
      error: null,
      videoSrc: sunVideo, // Video por defecto (soleado)
    };
  },
  methods: {
    async getWeather() {
      try {
        this.error = null;
        const apiKey = import.meta.env.VITE_OPENWEATHERMAP_API_KEY;
        if (!apiKey) {
          throw new Error('API key not found');
        }
        const response = await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(this.city)}&appid=${apiKey}&units=metric`);
        if (!response.ok) {
          throw new Error(`Error: ${response.status} ${response.statusText}`);
        }
        this.weather = await response.json();
        this.changeBackgroundVideo(); // Cambiar el video después de obtener el clima
      } catch (error) {
        this.error = error.message;
        console.error('Error fetching weather data:', error);
      }
    },
    changeBackgroundVideo() {
      if (!this.weather) return;

      const weatherMain = this.weather.weather[0].main.toLowerCase();

      // Cambiar la fuente del video según el clima
      switch (weatherMain) {
        case 'clear':
          this.videoSrc = sunVideo; // Día soleado
          break;
        case 'clouds':
          this.videoSrc = cloudsVideo; // Nublado
          break;
        case 'rain':
          this.videoSrc = rainVideo; // Lluvia
          break;
        case 'snow':
          this.videoSrc = snowVideo; // Nieve
          break;
        case 'wind':
        case 'thunderstorm':
          this.videoSrc = windVideo; // Viento
          break;
        default:
          this.videoSrc = sunVideo; // Video por defecto (soleado)
          break;
      }
    }
  }
};
</script>

<style scoped>
.background-video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: -1; 
    opacity: 0.7; 
  filter: blur(4px); 
}
.background-video::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.726); /* Capa semi-transparente */
  z-index: 1;
}
div {
  display: flex;
  flex-direction: column;
  align-items: center;
}
input {
  border: 1px solid #000000;
  font-family: 'Poppins', sans-serif;
  height: 30px;
  margin: 20px 0px 10px 0px;
}
h2, p {
  color: #000000;
  margin: 7px;
  font-family: 'Poppins', sans-serif;
}
input {
  color: #000000;
  padding: 2px;
  margin-right: 10px;
  transition: outline 0.01s ease-in-out;
}
span {
  font-weight: 600;
}
input:focus {
  outline: 2px solid #0077ff;
  border: 1px solid #00000000;
}
.error{
  color: #000;
}
.contentDiv {
  z-index: 3;
  height: 100vh;
  width: 400px;
  background-color: rgba(255, 255, 255, 0.664);
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
}
</style>
