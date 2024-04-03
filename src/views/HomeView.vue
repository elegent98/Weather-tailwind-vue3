<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        @input="getSearchResults"
        v-model="searchQuery"
        placeholder="Shahar yoki Mamlakatni kiriting..."
        type="text"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResult"
      >
        <p v-if="searchError">
          Xatolik yuz berdi.Iltimos takror urinib ko'ring
        </p>
        <p v-if="!serverError && mapboxSearchResult.length === 0">
          So'rovingizga mos natija topilmadi, boshqa atama kiriting
        </p>
        <template v-else>
          <li
            @click="previewCity(searchResult)"
            v-for="searchResult in mapboxSearchResult"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
    <Suspense>
      <CityList/>
      <template #fallback>
        <CityCardSkele/>
      </template>
    </Suspense>
    </div>
  </main>
</template>
<script setup>
import { ref } from "vue";
import axios from "axios";
import {useRouter} from "vue-router"
import CityList from "@/components/CityList.vue"
import CityCardSkele from "@/components/CityCardSkeleton.vue"

const router = useRouter()

const previewCity = (searchResult) =>{
    const [city,state] = searchResult.place_name.split(",")
    router.push({
        name:"cityView",
        params:{state: state.replaceAll(" ", "") , city:city},
        query:{
            lat:searchResult.geometry.coordinates[1],
            lng:searchResult.geometry.coordinates[0],
            preview:true
        }
    })
}
const mapboxSearchResult = ref(null);
const searchQuery = ref("");
const queryTimeout = ref(null);
const searchError = ref(null);
const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResult.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapboxSearchResult.value = null;
  }, 300);
};
</script>
<style></style>
