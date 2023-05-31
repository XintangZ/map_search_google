<template>
  <div class="input-group">
    <input
      type="text"
      class="form-control"
      placeholder="Search on the Map"
      v-model="query"
      @keyup.enter="searchQuery" />
    <button
      type="button"
      class="btn btn-outline-secondary"
      @click="searchQuery"
      :disabled="isSearching">
      <span
        v-if="isSearching"
        class="spinner-border spinner-border-sm"
        role="status"
        aria-hidden="true"></span>
      <i v-else class="fa-solid fa-magnifying-glass"></i>
    </button>
  </div>
</template>

<script>
import { onMounted, ref } from "vue";

export default {
  emits: ["resultsFound", "searchStatusChanged"],
  props: ["map", "isSearching"],
  setup(props, { emit }) {
    let service;
    const query = ref("");

    onMounted(async () => {
      // Import Google Places library
      const { PlacesService } = await google.maps.importLibrary("places");
      // Initialize Google Places
      service = new PlacesService(props.map);
    });

    const searchQuery = () => {
      emit("searchStatusChanged", true);
      // Handle empty query
      if (!query.value) {
        alert("Enter a search term.");
        emit("searchStatusChanged", false);
      } else {
        // Request config
        const request = {
          location: props.map.getCenter(),
          query: query.value,
          fields: ["name", "geometry"],
        };
        // Send request
        service.textSearch(request, (results, status, pagination) => {
          // Handle unsuccessful cases
          if (status !== "OK" || !results) {
            console.error("No result found.");
            emit("searchStatusChanged", false);

            return;
          }
          // Send results to App.vue
          emit("resultsFound", results);
          // Get more results (if any)
          if (pagination && pagination.hasNextPage) {
            pagination.nextPage();
          } else {
            emit("searchStatusChanged", false);
          }
        });
      }
    };

    return { query, searchQuery };
  },
};
</script>

<style></style>
