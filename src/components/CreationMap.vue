<template>
  <div id="blank"></div>
</template>
<script>
import L from "leaflet";
export default {
  props: ["dialogMap"],
  data: function () {
    return {
      map: L.Map,
    };
  },

  mounted() {
    this.initMap();
  },

  methods: {
    initMap() {
      if (document.getElementById("blank")) {
        console.log("entered dialog");
        document.getElementById("blank").setAttribute("id", "map");
      }
      this.map = L.map("map").setView([51.505, -0.09], 13);
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(this.map);
    },
  },

  computed: {},
  watch: {
    dialogMap(value) {
      console.log(value + "from dialog");
    },
  },
};
</script>

<style scoped>
#map {
  width: 50%;
  height: 250px;
}
</style>
