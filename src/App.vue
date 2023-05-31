<template>
  <div id="main" class="container-fluid mt-2 d-flex flex-column">
    <div class="row h-100">
      <div class="col-md-6">
        <div class="card">
          <div class="card-header p-3">
            <!-- LocationBtn and SearchBar -->
            <div class="row justify-content-between align-items-center">
              <!-- LocationBtn -->
              <div class="col-2 text-center">
                <LocationBtn @locationGot="onLocationGot" />
              </div>
              <!-- SearchBar -->
              <div class="col-10">
                <SearchBar
                  :map="map"
                  :isSearching="isSearching"
                  @resultsFound="onResultsFound"
                  @searchStatusChanged="toggleSearchStatus" />
              </div>
            </div>
          </div>
          <!-- Map -->
          <div class="card-body">
            <Map
              :userLocation="userLocation"
              :renderedResults="renderedResults"
              @mapLoaded="onMapLoaded" />
          </div>
        </div>
      </div>
      <div class="col-md-6 flex-wrap">
        <div class="card">
          <!-- Results -->
          <div class="card-body">
            <Results
              v-if="searchResults"
              :searchResults="searchResults"
              :isSearching="isSearching"
              @resultsRendered="onResultsRendered" />
            <div v-else class="h-100 position-relative">
              <p
                class="text-center text-secondary position-absolute top-50 start-50 translate-middle w-100">
                Use "<i class="fa-solid fa-location-crosshairs"></i>" To get
                current location.<br />
                Enter a search term. <br />
                Hit "Enter" or " <i class="fa-solid fa-magnifying-glass"></i>"
                to view results.
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

import Map from "./components/Map.vue";
import LocationBtn from "./components/LocationBtn.vue";
import SearchBar from "./components/SearchBar.vue";
import Results from "./components/Results.vue";

export default {
  name: "App",
  components: {
    Map,
    LocationBtn,
    SearchBar,
    Results,
  },
  setup() {
    const userLocation = ref(null);
    const map = ref(null);
    const searchResults = ref(null);
    const renderedResults = ref(null);
    const isSearching = ref(false);

    // Get user location from LocationBtn.vue, pass it to Map.vue to update map center
    const onLocationGot = location => {
      userLocation.value = location;
    };

    // Get map from Map.vue, pass it to SearchBar.vue to initialize Google Places
    const onMapLoaded = mapObj => {
      map.value = mapObj;
    };

    // Get latest results from SearchBar.vue, pass them to Results.vue to render
    const onResultsFound = results => {
      searchResults.value = results;
    };

    // Set search status
    const toggleSearchStatus = bool => {
      isSearching.value = bool;
      // Clear array when new search starts
      if (bool) {
        searchResults.value = [];
      }
    };

    // Get rendered results from Results.vue, pass them to Map.vue to add markers
    const onResultsRendered = results => {
      renderedResults.value = results;
    };

    return {
      userLocation,
      onLocationGot,
      onMapLoaded,
      map,
      onResultsFound,
      searchResults,
      toggleSearchStatus,
      onResultsRendered,
      renderedResults,
      isSearching,
    };
  },
};
</script>

<style>
#main {
  height: 98vh;
}
.card {
  min-height: 500px;
  min-width: 320px;
  height: 99%;
  width: 100%;
}
</style>
