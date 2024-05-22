<template>
  <v-card id="geotag-container" color="surface-light">
    <div id="map"></div>
    <v-text-field
      label="Latitude"
      variant="outlined"
      density="compact"
      max-width="30%"
    ></v-text-field>
    <v-text-field
      label="Longitude"
      variant="outlined"
      density="compact"
      max-width="30%"
    ></v-text-field>
    <!-- Map creation dialog -->
    <v-dialog v-model="dialogMap" max-width="500">
      <template v-slot:activator="{ props: activatorProps }">
        <v-btn
          v-bind="activatorProps"
          color="surface-variant"
          text="Open Dialog"
          variant="flat"
        ></v-btn>
      </template>
      <v-card title="Dialog">
        <CreationMap :dialogMap="dialogMap"></CreationMap>
        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn text="Close Dialog" @click="dialogMap = false"></v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
import CreationMap from "./components/CreationMap.vue";
import L from "leaflet";
export default {
  name: "MapComponent",
  components: {
    CreationMap,
  },
  data: function () {
    return {
      dialogMap: false,
      map: L.map,
    };
  },

  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      console.log("entered main page");
      this.map = L.map("map").setView([51.505, -0.09], 13);

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(this.map);
    },

    clearMap() {
      this.map != undefined ? this.map.remove() : null;
    },
  },

  computed: {},
  watch: {
    dialogMap(value) {
      value && document.getElementById("map").setAttribute("id", "wait");
      !value && document.getElementById("wait").setAttribute("id", "map");
    },
  },
};
</script>
<style scoped>
#map {
  width: 50%;
  height: 250px;
}
#wait {
  width: 50%;
  height: 250px;
  border: solid 1px;
}
</style>
