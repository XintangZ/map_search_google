<template>
  <div id="map"></div>
</template>

<script>
import { ref, onMounted, toRaw, watch, toRef } from "vue";

export default {
  emits: ["mapLoaded"],
  props: ["userLocation", "renderedResults"],
  setup(props, { emit }) {
    const defaultMapOptions = {
      center: props.userLocation
        ? props.userLocation
        : { lat: 56.1304, lng: -106.3468 },
      zoom: props.userLocation ? 12 : 4,
      disableDefaultUI: true,
      zoomControl: true,
      fullscreenControl: true,
    };
    const markers = ref([]);
    const userMarker = ref(null);

    onMounted(async () => {
      // Import map library
      const { Map } = await google.maps.importLibrary("maps");
      // Initialize map
      const map = new Map(document.getElementById("map"), defaultMapOptions);
      // Emit an event to indicate that the map has been loaded
      emit("mapLoaded", map);

      // Update map when user location changes
      const userLocation = toRef(props, "userLocation");
      watch(userLocation, (newValue, oldValue) => {
        // Config a marker for user's location
        const dotIcon = {
          path: google.maps.SymbolPath.CIRCLE,
          fillColor: "#4285F4",
          fillOpacity: 1,
          scale: 6,
          strokeColor: "#D2E3FC",
          strokeWeight: 2,
        };
        // Remove existing user location marker (if any)
        if (userMarker.value) {
          userMarker.value.setMap(null);
        }
        // Add new marker
        userMarker.value = new google.maps.Marker({
          position: newValue,
          icon: dotIcon,
          map,
        });
        // Update map center and zoom level
        map.setZoom(14);
        map.panTo(newValue);
      });

      // Add a new marker to the map
      const addMarker = result => {
        const marker = new google.maps.Marker({
          position: result.geometry.location,
          map,
          title: result.name,
          animation: google.maps.Animation.DROP,
        });

        marker.set("id", result.place_id);
        marker.setMap(map);
        markers.value.push(marker);
      };

      // Remove an existing marker from the map
      const removeMarker = marker => {
        toRaw(marker).setMap(null);
      };

      // Update bounds based on markers
      const updateBounds = () => {
        const bounds = new google.maps.LatLngBounds();
        for (let i = 0, len = markers.value.length; i < len; i++) {
          bounds.extend(markers.value[i].getPosition());
        }
        map.fitBounds(bounds);
      };

      // Update markers when rendered results changes
      const renderedResults = toRef(props, "renderedResults");
      watch(renderedResults, (newValue, oldValue) => {
        if (markers.value.length) {
          // If an old marker is not for new result, remove the marker
          for (let i = markers.value.length - 1; i >= 0; i--) {
            let uselessMarker = true;
            for (let newV of newValue) {
              if (markers.value[i].id === newV.place_id) {
                uselessMarker = false;
                break;
              }
            }

            if (uselessMarker) {
              removeMarker(markers.value[i]);
              markers.value.splice(i, 1);
            }
          }
        }

        for (let newV of newValue) {
          let hasMarker = false;
          // If a new result does not have corresponding marker in the markers array, add a new marker
          for (let marker of markers.value) {
            if (newV.place_id === marker.id) {
              hasMarker = true;
              break;
            }
          }
          if (!hasMarker) {
            addMarker(newV);
          }
        }

        // Move map center to the frist newly rendered result
        if (newValue.length) {
          map.panTo(newValue[0].geometry.location);
          updateBounds();
        }
      });
    });
  },
};
</script>

<style scoped>
#map {
  width: 100%;
  height: 100%;
}
</style>
