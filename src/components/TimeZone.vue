<template>
  <p v-if="timeZone && localTime" class="text-secondary">
    {{ timeZone }}<br />{{ localTime }}
  </p>
</template>

<script>
import { ref, watchEffect } from "vue";

export default {
  props: ["latestResult"],
  setup(props) {
    const timeZone = ref("");
    const localTime = ref("");
    let intervalId;

    const getTime = async (lat, lng) => {
      try {
        // Get timestamp, convert milliseconds to seconds
        const currentDate = new Date();
        const timestamp = Math.floor(currentDate.getTime() / 1000);
        const { data } = await axios.get(
          `https://maps.googleapis.com/maps/api/timezone/json?location=${lat}%2C${lng}&timestamp=${timestamp}&key=AIzaSyDzz8yVY4DJscchOYJgv1wuRb44vp4YXnU`
        );

        timeZone.value = data.timeZoneName;
        calculateLocalTime(data);
      } catch (err) {
        console.error(err);
      }
    };

    // Calculate local time based on UTC
    const calculateLocalTime = data => {
      // Get current utc
      const currentUTC = new Date();
      let yy = currentUTC.getUTCFullYear();
      let mm = currentUTC.getUTCMonth();
      let dd = currentUTC.getUTCDate();
      let h = currentUTC.getUTCHours();
      let m = currentUTC.getUTCMinutes();
      let s = currentUTC.getUTCSeconds();

      // if the time zone is in Daylight Savings Time
      if (data.dstOffset) {
        h++;
      }
      // Convert the offset from UTC in seconds to hours, adjust hours
      h += data.rawOffset / 60 / 60;

      // Update the time on the page every second
      intervalId = setInterval(() => {
        s++;
        const date = new Date(yy, mm, dd, h, m, s);
        localTime.value = formatDisplay(date);
      }, 1000);
    };

    const formatDisplay = date => {
      const formattedDate = date.toLocaleString(undefined, {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        second: "numeric",
      });

      return formattedDate;
    };

    // Rerender when latest result changes
    watchEffect(() => {
      if (props.latestResult) {
        clearInterval(intervalId);
        let lat = props.latestResult.geometry.location.lat();
        let lng = props.latestResult.geometry.location.lng();
        getTime(lat, lng);
      }
    });

    return { timeZone, localTime };
  },
};
</script>

<style></style>
