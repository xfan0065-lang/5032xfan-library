<script setup>
import { ref } from 'vue'

// Open-Meteo is free and does not need an API key
const GEOCODING_URL = 'https://geocoding-api.open-meteo.com/v1/search'
const FORECAST_URL = 'https://api.open-meteo.com/v1/forecast'

const city = ref('Melbourne')
const weather = ref(null)
const errorMessage = ref('')
const loading = ref(false)

// short descriptions for the WMO weather codes returned by Open-Meteo
const weatherDescriptions = {
  0: 'Clear sky',
  1: 'Mainly clear',
  2: 'Partly cloudy',
  3: 'Overcast',
  45: 'Fog',
  48: 'Depositing rime fog',
  51: 'Light drizzle',
  53: 'Moderate drizzle',
  55: 'Dense drizzle',
  61: 'Slight rain',
  63: 'Moderate rain',
  65: 'Heavy rain',
  71: 'Slight snow',
  73: 'Moderate snow',
  75: 'Heavy snow',
  80: 'Rain showers',
  81: 'Moderate rain showers',
  82: 'Violent rain showers',
  95: 'Thunderstorm',
  96: 'Thunderstorm with hail',
  99: 'Thunderstorm with heavy hail'
}

const fetchWeather = async (latitude, longitude, placeName) => {
  const url = `${FORECAST_URL}?latitude=${latitude}&longitude=${longitude}` +
    '&current=temperature_2m,apparent_temperature,relative_humidity_2m,weather_code,wind_speed_10m' +
    '&timezone=auto'

  const response = await fetch(url)
  if (!response.ok) {
    throw new Error('Weather service is not available right now.')
  }

  const data = await response.json()
  weather.value = {
    place: placeName,
    temperature: Math.round(data.current.temperature_2m),
    feelsLike: Math.round(data.current.apparent_temperature),
    humidity: data.current.relative_humidity_2m,
    windSpeed: data.current.wind_speed_10m,
    description: weatherDescriptions[data.current.weather_code] || 'Unknown',
    updatedAt: data.current.time.replace('T', ' ')
  }
}

const getWeather = async () => {
  errorMessage.value = ''
  weather.value = null

  if (!city.value.trim()) {
    errorMessage.value = 'Please enter a city name.'
    return
  }

  loading.value = true
  try {
    const response = await fetch(
      `${GEOCODING_URL}?name=${encodeURIComponent(city.value.trim())}&count=1&language=en&format=json`
    )
    const data = await response.json()

    if (!data.results || data.results.length === 0) {
      errorMessage.value = `Could not find a city called "${city.value}".`
      return
    }

    const place = data.results[0]
    const placeName = [place.name, place.admin1, place.country].filter(Boolean).join(', ')
    await fetchWeather(place.latitude, place.longitude, placeName)
  } catch (error) {
    errorMessage.value = error.message || 'Something went wrong while getting the weather.'
  } finally {
    loading.value = false
  }
}

const useMyLocation = () => {
  errorMessage.value = ''

  if (!navigator.geolocation) {
    errorMessage.value = 'Geolocation is not supported by this browser.'
    return
  }

  loading.value = true
  navigator.geolocation.getCurrentPosition(
    async (position) => {
      try {
        const { latitude, longitude } = position.coords
        await fetchWeather(latitude, longitude, 'My current location')
      } catch (error) {
        errorMessage.value = error.message
      } finally {
        loading.value = false
      }
    },
    () => {
      errorMessage.value = 'Location permission was denied.'
      loading.value = false
    }
  )
}
</script>

<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <h1 class="text-center">Weather Check</h1>
        <p class="text-center">Search for a city to see its current weather.</p>

        <form class="row g-2 mb-4" @submit.prevent="getWeather">
          <div class="col-sm-8">
            <label for="city" class="visually-hidden">City</label>
            <input
              id="city"
              v-model="city"
              type="text"
              class="form-control"
              placeholder="Enter a city, e.g. Melbourne"
            >
          </div>
          <div class="col-sm-4 d-grid">
            <button type="submit" class="btn btn-primary" :disabled="loading">
              {{ loading ? 'Loading...' : 'Get Weather' }}
            </button>
          </div>
        </form>

        <div class="text-center mb-4">
          <button type="button" class="btn btn-outline-secondary btn-sm" :disabled="loading" @click="useMyLocation">
            Use my location
          </button>
        </div>

        <div v-if="errorMessage" class="alert alert-danger">{{ errorMessage }}</div>

        <div v-if="weather" class="card shadow-sm">
          <div class="card-body text-center">
            <h2 class="h4">{{ weather.place }}</h2>
            <p class="display-4 mb-0">{{ weather.temperature }}&deg;C</p>
            <p class="lead">{{ weather.description }}</p>

            <div class="row">
              <div class="col">
                <strong>Feels like</strong>
                <p>{{ weather.feelsLike }}&deg;C</p>
              </div>
              <div class="col">
                <strong>Humidity</strong>
                <p>{{ weather.humidity }}%</p>
              </div>
              <div class="col">
                <strong>Wind</strong>
                <p>{{ weather.windSpeed }} km/h</p>
              </div>
            </div>

            <small class="text-muted">Local time of reading: {{ weather.updatedAt }}</small>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
