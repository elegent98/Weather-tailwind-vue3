<template>
  <Suspense>
    <template #default>
      <AsyncCityView />
    </template>
    <template #fallback>
      <CityViewSkeleton/>
    </template>
  </Suspense>
</template>
<script setup>
import AsyncCityView from "./AsyncCityView.vue";
import CityViewSkeleton from "@/components/CityViewSkeleton.vue";
</script>
<style></style>
