<template>
  <v-card color="surface-light" class="d-flex flex-column ga-2 ma-3 pa-3">
    <div>{{ $t("script.select-location") }}</div>
    <div id="blank"></div>
    <v-text-field
      id="latitude"
      v-model="latitude"
      label="Latitude"
      variant="outlined"
      density="compact"
    ></v-text-field>
    <v-text-field
      id="longitude"
      v-model="longitude"
      label="Longitude"
      variant="outlined"
      density="compact"
    ></v-text-field>
  </v-card>
</template>
<script>
import L from "leaflet";
export default {
  props: ["dialogMap"],
  data: function () {
    return {
      map: L.Map,
      marker: L.Marker,
      markerLayer: L.LayerGroup,
      latitude: "",
      longitude: "",
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
      this.markerLayer = new L.LayerGroup();
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(this.map);
      this.map.on("click", this.onMapClick);
      this.markerLayer.addTo(this.map);
    },
    onMapClick(e) {
      if (this.markerLayer.getLayers().length >= 1) {
        this.markerLayer.clearLayers();

        this.getLatLng(e);
      } else {
        this.getLatLng(e);
      }
    },

    getLatLng(e) {
      console.log(e.latlng);
      this.marker = L.marker(e.latlng).addTo(this.markerLayer);
      this.latitude = e.latlng.lat;
      this.longitude = e.latlng.lng;

      const geoLocation = {
        latitude: this.latitude,
        longitude: this.longitude,
      };

      sessionStorage.setItem("geoLocation", JSON.stringify(geoLocation));
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
  height: 250px;
}
</style>
