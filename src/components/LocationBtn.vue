<template>
  <button
    type="button"
    class="btn btn-primary"
    @click="getLocation"
    :disabled="isLoading">
    <span
      v-if="isLoading"
      class="spinner-border spinner-border-sm"
      role="status"
      aria-hidden="true"></span>
    <i v-else class="fa-solid fa-location-crosshairs"></i>
  </button>
</template>

<script>
import { ref } from "vue";

export default {
  emits: ["locationGot"],
  setup(_, { emit }) {
    const isLoading = ref(false);
    // Get user location
    const getLocation = e => {
      e.preventDefault();
      // Prevent multiple clicks while loading
      if (isLoading.value) {
        return;
      }

      // Set spinner
      isLoading.value = true;
      // Check if geolocation is supported
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          position => {
            const pos = {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            };
            // Pass the location to App.vue
            emit("locationGot", pos);
            isLoading.value = false;
          },
          error => {
            console.error(error.message);
            isLoading.value = false;
          }
        );
      } else {
        // Geolocation is not supported
        console.log("Geolocation is not supported by this browser.");
        isLoading.value = false;
      }
    };

    return { getLocation, isLoading };
  },
};
</script>

<style></style>
