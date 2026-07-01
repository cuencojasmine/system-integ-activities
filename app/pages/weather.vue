<template>
  <div class="page-bg">
    <div class="overlay">
      <div class="weather-card" v-if="currentWeather">

        <!-- Location -->
        <div class="location-block">
          <h2 class="city">
            {{ currentWeather?.location?.name }}<span class="region" v-if="currentWeather?.location?.region">, {{ currentWeather?.location?.region }}</span>
          </h2>
          <p class="country">{{ currentWeather?.location?.country }}</p>
        </div>

        <!-- Current conditions -->
        <div class="current-block">
          <img
            v-if="currentWeather?.current?.condition?.icon"
            class="current-icon"
            :src="`https:${currentWeather.current.condition.icon}`"
            :alt="currentWeather?.current?.condition?.text"
          />
          <div class="temp-block">
            <h3 class="temp">{{ Math.round(currentWeather?.current?.temp_c) }}°C</h3>
            <p class="condition">{{ currentWeather?.current?.condition?.text }}</p>
          </div>
        </div>

        <!-- Hourly forecast -->
        <section class="forecast-section">
          <h4 class="section-title">Hourly Forecast</h4>
          <div class="hourly-scroll">
            <div
              class="hour-chip"
              v-for="hour in upcomingHours"
              :key="hour.time_epoch"
            >
              <p class="hour-time">{{ formatHour(hour.time) }}</p>
              <img
                class="hour-icon"
                :src="`https:${hour.condition.icon}`"
                :alt="hour.condition.text"
              />
              <p class="hour-temp">{{ Math.round(hour.temp_c) }}°</p>
            </div>
          </div>
        </section>

        <!-- 7-day forecast -->
        <section class="forecast-section">
          <h4 class="section-title">7-Day Forecast</h4>
          <div class="daily-list">
            <div
              class="day-row"
              v-for="day in currentWeather?.forecast?.forecastday"
              :key="day.date_epoch"
            >
              <p class="day-name">{{ formatDay(day.date) }}</p>
              <img
                class="day-icon"
                :src="`https:${day.day.condition.icon}`"
                :alt="day.day.condition.text"
              />
              <p class="day-condition">{{ day.day.condition.text }}</p>
              <p class="day-temps">
                <span class="day-max">{{ Math.round(day.day.maxtemp_c) }}°</span>
                <span class="day-min">{{ Math.round(day.day.mintemp_c) }}°</span>
              </p>
            </div>
          </div>
        </section>

      </div>

      <!-- Loading state -->
      <div class="weather-card loading" v-else>
        <p>Loading weather…</p>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
//@ts-nocheck

const currentWeather = ref(null);

const getWeatherData = async () => {
  const data = await $fetch('https://api.weatherapi.com/v1/forecast.json?key=2ca1945da9c2479991832325262906&q=Manila&days=7&aqi=no&alerts=no')
  currentWeather.value = data

  
}

onMounted(getWeatherData);

const upcomingHours = computed(() => {
  if (!currentWeather.value?.forecast?.forecastday) return [];

  const allHours = currentWeather.value.forecast.forecastday.flatMap(
    (d) => d.hour
  );
  const nowEpoch = currentWeather.value.location?.localtime_epoch ?? Date.now() / 1000;

  return allHours.filter((h) => h.time_epoch >= nowEpoch).slice(0, 12);
});

const formatHour = (timeStr) => {
  const date = new Date(timeStr.replace(' ', 'T'));
  return date.toLocaleTimeString([], { hour: 'numeric', hour12: true });
};

const formatDay = (dateStr) => {
  const date = new Date(dateStr + 'T00:00:00');
  const today = new Date();
  const isToday = date.toDateString() === today.toDateString();
  return isToday ? 'Today' : date.toLocaleDateString([], { weekday: 'short' });
};
</script>

<style>
.page-bg {
  background-image: url(https://4kwallpapers.com/images/wallpapers/milky-way-starry-sky-night-mountains-lake-reflection-cold-5k-3440x1440-287.jpg);
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  height: 100vh;
  width: 100%;
}

.overlay {
  height: 100%;
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 48px 16px;
  box-sizing: border-box;
  overflow-y: auto;
  background: linear-gradient(180deg, rgba(5, 8, 20, 0.35) 0%, rgba(5, 8, 20, 0.55) 100%);
}

.weather-card {
  width: 100%;
  max-width: 420px;
  color: #f5f6fa;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: rgba(20, 24, 38, 0.45);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 24px;
  padding: 28px 24px 32px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.45);
}

.weather-card.loading {
  text-align: center;
  padding: 60px 24px;
  font-size: 16px;
  color: rgba(245, 246, 250, 0.8);
}

/* Location */
.location-block {
  text-align: center;
  margin-bottom: 18px;
}

.city {
  font-size: 26px;
  font-weight: 600;
  margin: 0;
  letter-spacing: 0.2px;
}

.region {
  font-weight: 400;
  opacity: 0.85;
}

.country {
  margin: 4px 0 0;
  font-size: 14px;
  font-weight: 400;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  opacity: 0.65;
}

/* Current */
.current-block {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 12px 0 24px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.12);
  margin-bottom: 22px;
}

.current-icon {
  width: 72px;
  height: 72px;
  filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.3));
}

.temp-block {
  text-align: left;
}

.temp {
  font-size: 52px;
  font-weight: 300;
  margin: 0;
  line-height: 1;
}

.condition {
  margin: 4px 0 0;
  font-size: 15px;
  opacity: 0.85;
}

/* Section headers */
.section-title {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 1.2px;
  text-transform: uppercase;
  opacity: 0.7;
  margin: 0 0 12px;
}

.forecast-section {
  margin-bottom: 24px;
}

.forecast-section:last-child {
  margin-bottom: 0;
}

/* Hourly */
.hourly-scroll {
  display: flex;
  gap: 14px;
  overflow-x: auto;
  padding-bottom: 4px;
  scrollbar-width: thin;
}

.hourly-scroll::-webkit-scrollbar {
  height: 4px;
}

.hourly-scroll::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.25);
  border-radius: 4px;
}

.hour-chip {
  flex: 0 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  min-width: 56px;
  padding: 10px 6px;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.06);
}

.hour-time {
  font-size: 12px;
  opacity: 0.75;
  margin: 0;
  white-space: nowrap;
}

.hour-icon {
  width: 32px;
  height: 32px;
}

.hour-temp {
  font-size: 14px;
  font-weight: 600;
  margin: 0;
}

/* Daily */
.daily-list {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.day-row {
  display: grid;
  grid-template-columns: 56px 32px 1fr auto;
  align-items: center;
  gap: 12px;
  padding: 10px 6px;
  border-radius: 12px;
}

.day-row:hover {
  background: rgba(255, 255, 255, 0.05);
}

.day-name {
  font-size: 14px;
  font-weight: 500;
  margin: 0;
}

.day-icon {
  width: 28px;
  height: 28px;
}

.day-condition {
  font-size: 13px;
  opacity: 0.75;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.day-temps {
  margin: 0;
  font-size: 14px;
  font-weight: 500;
  display: flex;
  gap: 8px;
  justify-self: end;
}

.day-min {
  opacity: 0.55;
}

@media (max-width: 480px) {
  .weather-card {
    max-width: 100%;
    padding: 24px 18px 28px;
  }
  .temp {
    font-size: 44px;
  }
}
</style>