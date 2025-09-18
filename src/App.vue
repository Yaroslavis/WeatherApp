<template>
  <div class="weather" :class="weatherClass">
    <div class="container">
      <!-- Поисковая форма -->
      <div class="card weather-form">
        <input 
          type="text" 
          class="weather-form_input"
          v-model="searchQuery"
          @keyup.enter="weatherSearch"
          placeholder="Введите город"> 
        <button class="weather-form_btn" @click="weatherSearch">
          <i class="fas fa-search"></i> Поиск
        </button>
        <div class="mobile-keyboard-hint">Нажмите Enter на клавиатуре</div>
      </div>

      <!-- Загрузка -->
      <div class="card weather-load" v-if="loading">
        <i class="fas fa-spinner fa-spin"></i> Загрузка...
      </div>

      <!-- Ошибка -->
      <div class="card" v-if="error">
        <i class="fas fa-exclamation-circle"></i> Ошибка: Город не найден
      </div>

      <!-- Результаты -->
      <div class="weather-info" v-if="!error && !loading && location && temperature !== null && description">
        <div class="weather-info_text">
          <p class="card city">{{ location }}</p>
          <p class="card temp">{{ temperature }}°C</p>
          <p class="card status">
            <img v-if="icon" :src="icon" alt="icon" class="weather-icon" />
            {{ description }}
          </p>
        </div>
      </div>
    </div>

    <!-- Фон -->
    <div class="weather-bg">
      <img class="weather-bg_img bg" src="https://images.unsplash.com/photo-1506744038136-46273834b3fb" alt="Фон по умолчанию">
      <img class="weather-bg_img sunny" :src="backgroundImages.sunny" alt="Солнечно">
      <img class="weather-bg_img overcast" :src="backgroundImages.overcast" alt="Пасмурно">
      <img class="weather-bg_img partly-cloudy" :src="backgroundImages.partlyCloudy" alt="Переменная облачность">
      <img class="weather-bg_img rainy" :src="backgroundImages.rainy" alt="Дождь">
      <img class="weather-bg_img snowy" :src="backgroundImages.snowy" alt="Снег">
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  name: 'App',
  setup() {
    const location = ref('');
    const temperature = ref(null);
    const description = ref('');
    const icon = ref('');
    const loading = ref(false);
    const error = ref(false);
    const searchQuery = ref('');

    const unsplashAccessKey = 'eYvBpRAgCOEcrWOh7i6BDuU8YBI2BJId_KnKkNpSQ-o';

    const backgroundImages = ref({
      sunny: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4',
      overcast: 'https://images.unsplash.com/photo-1518837695005-2083093ee35b',
      partlyCloudy: 'https://images.unsplash.com/photo-1516466723877-e4ec1d736c8a',
      rainy: 'https://images.unsplash.com/photo-1433863448220-9aaa1e35eccb',
      snowy: 'https://images.unsplash.com/photo-1542601098-8fc114e148e2'
    });

    const weatherClass = computed(() => {
      const desc = description.value.toLowerCase();
      if (desc.includes('sunny') || desc.includes('ясно')) return 'sunny';
      if (desc.includes('overcast') || desc.includes('пасмурно')) return 'overcast';
      if (desc.includes('partly cloudy') || desc.includes('переменная облачность')) return 'partly-cloudy';
      if (desc.includes('rain') || desc.includes('дождь')) return 'rainy';
      if (desc.includes('snow') || desc.includes('снег')) return 'snowy';
      return '';
    });

    async function fetchBackgroundImage(query) {
      try {
        const response = await fetch(
          `https://api.unsplash.com/photos/random?query=${query}&orientation=landscape&client_id=${unsplashAccessKey}`
        );
        if (!response.ok) throw new Error('Ошибка загрузки фона');
        const data = await response.json();
        return data.urls.regular;
      } catch (err) {
        console.error('Ошибка при загрузке фона:', err);
        return null;
      }
    }

    async function weatherSearch() {
      if (!searchQuery.value.trim()) return;

      loading.value = true;
      error.value = false;

      try {
        const response = await fetch(
          `https://api.weatherapi.com/v1/current.json?key=cb4f6aa038ad4842abe81432251709&q=${searchQuery.value}&lang=ru`
        );
        if (!response.ok) throw new Error("Город не найден");

        const data = await response.json();
        location.value = data.location.name;
        temperature.value = data.current.temp_c;
        description.value = data.current.condition.text;
        icon.value = `https:${data.current.condition.icon}`;

        // Определяем запрос для фона
        let backgroundQuery = 'weather';
        const desc = description.value.toLowerCase();
        if (desc.includes('sunny') || desc.includes('ясно')) backgroundQuery = 'sunny+landscape';
        else if (desc.includes('overcast') || desc.includes('пасмурно')) backgroundQuery = 'cloudy+sky';
        else if (desc.includes('partly cloudy') || desc.includes('переменная облачность')) backgroundQuery = 'partly+cloudy+sky';
        else if (desc.includes('rain') || desc.includes('дождь')) backgroundQuery = 'rainy+weather';
        else if (desc.includes('snow') || desc.includes('снег')) backgroundQuery = 'snow+landscape';

        const newBackground = await fetchBackgroundImage(backgroundQuery);
        if (newBackground) {
          switch(weatherClass.value) {
            case 'sunny': backgroundImages.value.sunny = newBackground; break;
            case 'overcast': backgroundImages.value.overcast = newBackground; break;
            case 'partly-cloudy': backgroundImages.value.partlyCloudy = newBackground; break;
            case 'rainy': backgroundImages.value.rainy = newBackground; break;
            case 'snowy': backgroundImages.value.snowy = newBackground; break;
          }
        }

      } catch (err) {
        error.value = true;
        console.error(err);
      } finally {
        loading.value = false;
        searchQuery.value = '';
      }
    }

    return {
      location,
      temperature,
      description,
      icon,
      loading,
      error,
      searchQuery,
      backgroundImages,
      weatherClass,
      weatherSearch
    };
  }
}
</script>