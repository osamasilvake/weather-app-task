<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import type { WeatherData } from "./WeatherApp.interface";

const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;
const API_URL = import.meta.env.VITE_API_BASE_URL;

const city = ref("");
const weather = ref<WeatherData | null>(null);
const loading = ref(false);
const error = ref("");

const fetchWeather = async () => {
  if (!city.value.trim()) return;

  loading.value = true;
  error.value = "";
  weather.value = null;

  try {
    const { data } = await axios.get<WeatherData>(API_URL, {
      params: {
        q: city.value,
        appid: API_KEY,
        units: "metric",
      },
    });

    weather.value = data;
  } catch (err) {
    error.value = "City not found, please try again.";
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <div class="weather-app">
    <div class="weather-app__container">
      <h1 class="weather-app__title">Weather Forecast</h1>

      <div class="weather-app__search-bar">
        <input
          v-model="city"
          class="weather-app__search-bar-input"
          placeholder="Enter city..."
          @keyup.enter="fetchWeather"
        />
        <button class="weather-app__search-bar-button" @click="fetchWeather">
          Search
        </button>
      </div>

      <div v-if="loading" class="weather-app__loading">Loading...</div>
      <div v-if="error" class="weather-app__error">{{ error }}</div>

      <transition name="fade">
        <div v-if="weather" class="weather-app__weather-card">
          <h2 class="weather-app__city">{{ weather.name }}</h2>
          <img
            :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@4x.png`"
            alt="weather icon"
            class="weather-app__icon"
          />
          <p class="weather-app__temperature">{{ weather.main.temp }}°C</p>
          <p class="weather-app__description">
            {{ weather.weather[0].description }}
          </p>
          <p class="weather-app__humidity">
            Humidity: {{ weather.main.humidity }}%
          </p>
          <p class="weather-app__wind">Wind: {{ weather.wind.speed }} km/h</p>
        </div>
      </transition>
    </div>
  </div>
</template>

<style lang="scss" scoped>
$primary-bg: linear-gradient(135deg, #e3f2fd, #bbdefb);
$container-bg: rgba(255, 255, 255, 0.6);
$text-color: #2a2a2a;
$border-radius: 12px;
$shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);

.weather-app {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: $primary-bg;
  font-family: "Inter", sans-serif;
  color: $text-color;

  &__container {
    background: $container-bg;
    padding: 20px;
    border-radius: $border-radius;
    text-align: center;
    width: 360px;
    backdrop-filter: blur(10px);
    box-shadow: $shadow;
    animation: slideDown 0.5s ease-in-out;

    @keyframes slideDown {
      from {
        transform: translateY(-20px);
        opacity: 0;
      }
      to {
        transform: translateY(0);
        opacity: 1;
      }
    }
  }

  &__title {
    font-size: 24px;
    font-weight: bold;
  }

  &__search-bar {
    display: flex;
    gap: 10px;
    margin: 20px 0;

    &-input {
      flex: 1;
      padding: 10px;
      border: none;
      border-radius: 8px;
      font-size: 16px;
      background: rgba(255, 255, 255, 0.8);
      outline: none;
      color: $text-color;

      &::placeholder {
        color: darken($text-color, 20%);
      }
    }

    &-button {
      padding: 10px 15px;
      background: #ffcc00;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
      transition: all 0.3s ease;

      &:hover {
        background: #e6b800;
        transform: scale(1.05);
      }
    }
  }

  &__weather-card {
    background: rgba(255, 255, 255, 0.55);
    padding: 20px;
    border-radius: $border-radius;
    margin-top: 15px;
    box-shadow: 0px 3px 8px rgba(0, 0, 0, 0.1);
    animation: fadeIn 0.8s ease-in-out;

    @keyframes fadeIn {
      from {
        opacity: 0;
        transform: scale(0.95);
      }
      to {
        opacity: 1;
        transform: scale(1);
      }
    }
  }

  &__temperature {
    font-size: 32px;
    font-weight: bold;
    margin-top: 10px;
  }

  &__description {
    text-transform: capitalize;
    font-size: 18px;
    margin-bottom: 10px;
  }

  &__humidity,
  &__wind {
    font-size: 14px;
  }
}

/* Fade Transition */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease-in-out;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
