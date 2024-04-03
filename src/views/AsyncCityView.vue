<template>
  <div class="flex flex-col flex-1 items-center">
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        “+” belgisini bosish orqali siz bu shaharni doimiy kuzatish imkoniyatiga
        ega bo'lasiz
      </p>
    </div>
    <!-- Ob-havo haqida umumiy ma'lumot -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleString("en-us", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          new Date(weatherData.currentTime).toLocaleString("en-us", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round((Math.round(weatherData.current.temp)-32)*5/9)}}&deg;
      </p>

      <p>Taxminan {{ Math.round((Math.round(weatherData.current.feels_like)-32)*5/9)}}&deg;</p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="`https://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`"
      />
    </div>
    <hr class="border-white border border-opacity-10 w-full" />
    <!-- Soatlik ob-havo -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Soatlik ob-havo</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"
              alt=""
            />
            <p class="text-xl">{{ Math.round((Math.round(hourData.temp)-32)*5/9)}}&deg;</p>
          </div>
        </div>
      </div>
    </div>
    <hr class="border-white border-opacity-10 boder w-full ">
    <!-- Haftalik ob-havo -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="m-4 text-center text-2xl"> 7 Kunlik ob-havo </h2>
        <div class="flex items-center" v-for="day in weatherData.daily" :key="day.dt">
          <p class="flex-1">
            {{ new Date(day.dt*1000).toLocaleDateString(
              "en-us",
              {
                weekday:"long"
              }
            ) }}
          </p>
          <img 
          class="w-[50px] h-[50px] object-cover" 
          :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`">
          <div class="flex  gap-3  flex-1 justify-end">
          <p> Max:{{ Math.round((Math.round(day.temp.max)-32)*5/9) }}&deg; </p>
          <p> Min:{{ Math.round((Math.round(day.temp.min)-32)*5/9) }}&deg; </p> 
          </div>
        </div>
      </div>
    </div>
    <!-- //! Saqlangan shaharni o'chirish     -->
    <div @click="removeCity" class="flex items-center gap-2 py-5 text-xl text-white cursor-pointer duration-150 hover:text-red-600"> 
      <i class="fa-solid fa-trash"></i>
      <p > Olib tashlash </p> 
    </div>

 </div>
</template>
<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });
    //Yuklanish

    await new Promise((res) => setTimeout(res,1000) )

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();

const router = useRouter()
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"))
  const updateCities = cities.filter(
    (city) => {
      city.id !== route.query.id
    }
  )
  localStorage.setItem("savedCities", JSON.stringify(updateCities))

  router.push({
    name:'home'
  })
}

</script>
