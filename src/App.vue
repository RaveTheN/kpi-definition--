<template>
  <v-row>
    <v-col cols="12">
      <!-- Upper Geotag container -->
      <v-card id="geotag-container" color="surface-light">
        <div id="map"></div>
        <v-text-field
          label="Latitude"
          v-model="latitude"
          variant="outlined"
          density="compact"
          max-width="30%"
        ></v-text-field>
        <v-text-field
          label="Longitude"
          v-model="longitude"
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

              <v-btn text="Close Dialog" @click="onMarkerDialogClose"></v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- END Map creation dialog -->
      </v-card>
      <!-- END Geotag container -->
    </v-col>
  </v-row>
  <v-row>
    <v-col cols="8">
      <!-- Measure and KPI selection -->
      <v-card>
        <v-tabs v-model="tab" bg-color="primary">
          <v-tab value="one">Item One</v-tab>
          <v-tab value="two">Item Two</v-tab>
        </v-tabs>

        <v-tabs-window v-model="tab">
          <v-tabs-window-item value="one">
            <v-card>
              <v-container>
                <v-row justify="start">
                  <v-col v-if="!allSelected" cols="12">
                    <v-text-field
                      ref="searchField"
                      v-model="search"
                      label="Search"
                      hide-details
                      single-line
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>

              <v-divider v-if="!allSelected"></v-divider>

              <v-virtual-scroll :items="searchResult" height="100%">
                <template v-slot:default="{ item }">
                  <v-list-item
                    v-if="!selected.includes(item)"
                    :key="item.text"
                    :disabled="loading"
                    @click="selected.push(item)"
                  >
                    <v-list-item-title v-text="item.text"></v-list-item-title>
                  </v-list-item>
                </template>
              </v-virtual-scroll>

              <v-divider></v-divider>
            </v-card>
          </v-tabs-window-item>
          <v-tabs-window-item value="two"> Two </v-tabs-window-item>
        </v-tabs-window>
      </v-card>
    </v-col>
    <v-col cols="4">
      <!-- END Measure and KPI selection -->
      <v-card>
        <v-container>
          <div height="10%" bg-color="primary">
            <h2>Selected</h2>
            <v-list
              ><v-list-item
                v-for="(selection, i) in selections"
                :key="selection.text"
              >
                {{ selection.text }}
                <v-btn :disabled="loading" @click="selected.splice(i, 1)">
                  delete
                </v-btn>
              </v-list-item></v-list
            >
          </div>
        </v-container>
      </v-card>
      <v-spacer></v-spacer>

      <v-btn
        :disabled="!selected.length"
        :loading="loading"
        color="purple"
        variant="text"
        @click="next"
      >
        Next
      </v-btn>
    </v-col>
  </v-row>
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
      tab: null,
      items: [
        {
          text: "Nature",
          icon: "mdi-nature",
        },
        {
          text: "Nightlife",
          icon: "mdi-glass-wine",
        },
        {
          text: "November",
          icon: "mdi-calendar-range",
        },
        {
          text: "Portland",
          icon: "mdi-map-marker",
        },
        {
          text: "Biking",
          icon: "mdi-bike",
        },
      ],
      loading: false,
      search: "",
      selected: [],
      dialogMap: false,
      map: L.map,
      latitude: "",
      longitude: "",

      marker: L.Marker,
      markerLayer: L.LayerGroup,

      editedItem: {
        id: "",
        category: "",
        name: "",
        unitMeasure: "",
        lastValue: "",
        targetValue: "",
        owner: "",
        sharedWith: [],
        formula: {
          id: "6f422054-0dfd-4d4d-b611-8d255eb1f0c0",
          type: "defined",
          formulaDetails: [
            {
              measureId: "176171d3-5a37-49ec-90ff-9727e535b502",
              operation: "SUM(A1:A15)",
            },
            {
              measureId: "176171d3-5a37-49ec-90ff-9727e535b502",
              operation: "SUM(A1:A10)",
            },
          ],
        },
        geoLocation: {
          id: "b61c0f61-5671-448e-b951-9932a894a402",
          latitude: 34.0522,
          longitude: -118.2437,
        },
      },
      defaultItem: {
        id: "",
        category: "",
        name: "",
        unitMeasure: "",
        lastValue: "",
        targetValue: "",
        owner: "",
        sharedWith: [],
      },
    };
  },

  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      console.log("entered main page");
      this.map = L.map("map").setView([51.505, -0.09], 13);

      this.markerLayer = new L.LayerGroup();
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(this.map);

      this.markerLayer.addTo(this.map);
    },

    clearMap() {
      this.map != undefined ? this.map.remove() : null;
    },

    setLatLng(coordinates) {
      this.latitude = coordinates.latitude;
      this.longitude = coordinates.longitude;
    },

    setMarker(coordinates) {
      if (this.markerLayer.getLayers().length >= 1) {
        this.markerLayer.clearLayers();
      }
      this.marker = L.marker({
        lat: coordinates.latitude,
        lng: coordinates.longitude,
      }).addTo(this.markerLayer);
    },

    onMarkerDialogClose() {
      const coordinates = JSON.parse(sessionStorage.getItem("geoLocation"));
      this.dialogMap = false;
      this.setLatLng(coordinates);
      this.setMarker(coordinates);
    },

    next() {
      this.loading = true;

      setTimeout(() => {
        this.search = "";
        this.selected = [];
        this.loading = false;
      }, 2000);
    },
  },

  computed: {
    allSelected() {
      return this.selected.length === this.items.length;
    },
    searchResult() {
      const search = this.search.toLowerCase();

      if (!search) return this.items;

      return this.items.filter((item) => {
        //convert item to text and looks in it for matches - might consider replacing for the actual measures
        const text = item.text.toLowerCase();

        return text.indexOf(search) > -1;
      });
    },
    selections() {
      const selections = [];

      for (const selection of this.selected) {
        selections.push(selection);
      }

      return selections;
    },
  },
  watch: {
    dialogMap(value) {
      value && document.getElementById("map").setAttribute("id", "wait");
      !value && document.getElementById("wait").setAttribute("id", "map");
    },

    selected() {
      this.search = "";
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
