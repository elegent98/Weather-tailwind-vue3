<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6">
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-sun text-2xl"></i>
          <p class="text-2xl">Mahalliy ob-havo</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>
        <i
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="addCity"
          v-if="route.query.preview"
        ></i>
      </div>

      <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
        <div class="text-black">
          <h1 class="text-2xl mb-1">Web dastur haqida:</h1>
          <p class="mb-4">
            Mahalliy ob-havo sizga shaharlarning hozirgi va kelajakdagi ob-havosini
            kuzatish imkonini beradi sizning tanlovingiz bo'yicha.
          </p>
          <h2 class="text-2xl">Bu qanday ishlaydi:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>Qidiruv satriga nomni kiritish orqali shahringizni qidiring.</li>
            <li>
              Natijalar ichidan shaharni tanlang, bu sizni joriy ob-havoga olib boradi
              sizning tanlovingiz uchun.
            </li>
            <li>
              Yuqori o'ng burchakdagi "+" belgisini bosish orqali shaharni kuzatib borishingiz mumkin.
            </li>
          </ol>

          <h2 class="text-2xl">Saqlangan shaharni o'chirish</h2>
          <p>
            Agar siz endi shaharni kuzatishni xohlamasangiz, shunchaki bosh sahifa past qismidagi shaharni o'chirish varianti bo'ladi.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";



const savedCities = ref([]);
const route = useRoute();
const router = useRouter();
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem("savedCities", JSON.stringify(savedCities.value));

  let query = Object.assign({}, route.query);
  delete query.preview;
  query.id = locationObj.id;
  router.replace({ query });
};

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>
