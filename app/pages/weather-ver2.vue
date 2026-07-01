<template>
  <v-container class="d-flex justify-center align-center" style="min-height: 100vh;">
    <v-card width="400" v-if="currentWeather">

      <v-card-title>{{ currentWeather.location.name }}</v-card-title>
      <v-card-subtitle>{{ currentWeather.location.region }}</v-card-subtitle>
      <v-card-subtitle>{{ currentWeather.location.country }}</v-card-subtitle>

      <v-divider />

      <v-card-text>
        <v-row align="center">
          <v-col cols="auto">
            <v-img :src="`https:${currentWeather.current.condition.icon}`" width="64" />
          </v-col>
          <v-col>
            <div style="font-size: 48px;">{{ currentWeather.current.temp_c.toFixed(1) }}°C</div>
            <div>{{ currentWeather.current.condition.text }}</div>
            <div>Feels like {{ currentWeather.current.feelslike_c.toFixed(1) }}°C</div>
          </v-col>
        </v-row>
      </v-card-text>

      <v-divider />

      <v-card-text>
        <div><strong>Hourly Forecast</strong></div>
        <v-row>
          <v-col cols="auto" v-for="hour in upcomingHours" :key="hour.time_epoch">
            <div>{{ formatHour(hour.time) }}</div>
            <v-img :src="`https:${hour.condition.icon}`" width="32" />
            <div>{{ hour.temp_c.toFixed(1) }}°</div>
          </v-col>
        </v-row>
      </v-card-text>

      <v-divider />

      <!-- <v-card-text>
        <div><strong>7-Day Forecast</strong></div>
        <v-row v-for="day in currentWeather.forecast.forecastday" :key="day.date_epoch" align="center">
          <v-col cols="3">{{ formatDay(day.date) }}</v-col>
          <v-col cols="2"><v-img :src="`https:${day.day.condition.icon}`" width="28" /></v-col>
          <v-col>{{ day.day.condition.text }}</v-col>
          <v-col cols="auto">{{ day.day.maxtemp_c.toFixed(1) }}° / {{ day.day.mintemp_c.toFixed(1) }}°</v-col>
        </v-row>
      </v-card-text> -->

    </v-card>

    <v-card width="400" v-else>
      <v-card-text>Loading weather...</v-card-text>
    </v-card>
  </v-container>
</template>

<script lang="ts" setup>
//@ts-nocheck

const currentWeather = ref(null);
const upcomingHours = ref([]);

const getWeatherData = async () => {
  const data = await $fetch('https://api.weatherapi.com/v1/forecast.json?key=2ca1945da9c2479991832325262906&q=San Fernando&days=7&aqi=no&alerts=no')
  currentWeather.value = data

  const nowEpoch = data.location?.localtime_epoch ?? Date.now() / 1000;
  const todayHours = data.forecast.forecastday[0].hour;
  upcomingHours.value = todayHours.filter((h) => h.time_epoch >= nowEpoch);
}

const formatHour = (timeStr) => {
  const date = new Date(timeStr.replace(' ', 'T'));
  return date.toLocaleTimeString([], { hour: 'numeric', hour12: true });
};

// const formatDay = (dateStr) => {
//   const date = new Date(dateStr + 'T00:00:00');
//   const today = new Date();
//   const isToday = date.toDateString() === today.toDateString();
//   return isToday ? 'Today' : date.toLocaleDateString([], { weekday: 'short' });
// };

onMounted(getWeatherData);
</script>