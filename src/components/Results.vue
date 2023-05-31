<template>
  <div class="w-100 h-100 d-flex flex-column">
    <!-- List of results -->
    <div class="order-2 mt-3">
      <ul class="list-group overflow-auto">
        <li
          v-for="result in renderedResults"
          :key="result.place_id"
          class="list-group-item d-flex align-items-center">
          <input
            type="checkbox"
            class="form-check-input me-1"
            :id="result.place_id"
            @click="toggleSelection(result)" /><label
            class="form-check-label ms-2"
            :for="result.place_id"
            >{{ result.name }}</label
          >
        </li>
      </ul>
    </div>

    <!-- Paigination and DeleteBtn -->
    <div v-if="results.length" id="navs" class="order-1 justify-content-center">
      <div class="row mb-1">
        <!-- Paigination -->
        <div class="col-10">
          <div class="btn-group" role="group">
            <!-- Previous page btn -->
            <button
              type="button"
              class="btn btn-outline-primary"
              @click="prevPage"
              :disabled="currentPage === 1">
              &laquo;
            </button>
            <button class="btn btn-outline-primary" disabled>
              {{ currentPage }} / {{ totalPages }}
            </button>
            <!-- Next page btn -->
            <button
              type="button"
              class="btn btn-outline-primary"
              @click="nextPage"
              :disabled="currentPage === totalPages">
              &raquo;
            </button>
          </div>
        </div>
        <!-- DeleteBtn -->
        <div class="col-2 text-center">
          <button
            type="button"
            class="btn btn-danger"
            @click="deleteSelectedResults"
            :disabled="!selectedResults.length">
            <i class="fa-solid fa-trash-can"></i>
          </button>
        </div>
      </div>
    </div>
    <div
      v-else
      class="h-100 w-100 d-flex align-items-center justify-content-center">
      <p class="text-secondary">No Result</p>
    </div>
    <!-- TimeZone -->
    <div v-if="results.length && latestResult" class="order-3 mt-3 px-1">
      <TimeZone :latestResult="latestResult" />
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, toRef } from "vue";
import TimeZone from "./TimeZone.vue";

export default {
  emits: ["resultsRendered"],
  props: ["searchResults", "isSearching"],
  components: { TimeZone },
  setup(props, { emit }) {
    const pageSize = 10;
    const currentPage = ref(1);

    //Calculate total pages
    const totalPages = computed(() => {
      return Math.ceil(results.value.length / pageSize);
    });

    // Decide which results to render based on current page
    const renderedResults = computed(() => {
      const startIndex = (currentPage.value - 1) * pageSize;
      const endIndex = startIndex + pageSize;
      return results.value.slice(startIndex, endIndex);
    });

    // Make a copy array to store all results
    const results = ref([]);
    const latestResult = ref(null);
    const searchResults = toRef(props, "searchResults");
    watch(searchResults, (newValue, oldValue) => {
      // Clear old results from last search
      if (!oldValue.length) {
        results.value = [];
        latestResult.value = null;
        currentPage.value = 1;
      }
      // Add latest results to the copy array
      if (newValue.length) {
        results.value = [...results.value, ...newValue];
        latestResult.value = results.value[results.value.length - 1];
      }
    });

    // Send rendered results to App.vue, then pass them to Map.vue to add markers
    watch(renderedResults, (newValue, oldValue) => {
      if (newValue !== oldValue) {
        emit("resultsRendered", newValue);
      }
    });

    // Navigate to previous page (if available)
    const prevPage = e => {
      e.preventDefault();
      if (currentPage.value > 1) {
        currentPage.value--;
        clearSelected();
      }
    };
    // Navigate to next page (if available)
    const nextPage = e => {
      e.preventDefault();
      if (currentPage.value < totalPages.value) {
        currentPage.value++;
        clearSelected();
      }
    };

    const selectedResults = ref([]);
    const toggleSelection = result => {
      if (isSelected(result)) {
        selectedResults.value = selectedResults.value.filter(
          selectedResult => selectedResult !== result
        );
      } else {
        selectedResults.value.push(result);
      }
    };

    const isSelected = result => {
      return selectedResults.value.includes(result);
    };

    // Remove selected results from results array, clear selected array
    const deleteSelectedResults = () => {
      results.value = results.value.filter(
        result => !selectedResults.value.includes(result)
      );

      if (currentPage.value > totalPages.value) {
        currentPage.value = totalPages.value;
      }

      clearSelected();
    };

    const clearSelected = () => {
      selectedResults.value = [];
    };

    return {
      results,
      currentPage,
      totalPages,
      renderedResults,
      prevPage,
      nextPage,
      selectedResults,
      toggleSelection,
      deleteSelectedResults,
      latestResult,
    };
  },
};
</script>

<style scoped>
#navs {
  width: 95%;
}
</style>
