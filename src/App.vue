<template>
  <!-- Upper Geotag container -->
  <v-card
    id="geotag-container"
    color="surface-light"
    class="d-flex align center flex-row align-center ga-9 ma-2 pa-5"
  >
    <div id="map"></div>
    <div class="flex-grow-1">
      <v-text-field
        :label="$t('labels.latitude')"
        v-model="latitude"
        variant="outlined"
        density="compact"
        readonly
      ></v-text-field>
      <v-text-field
        :label="$t('labels.longitude')"
        v-model="longitude"
        variant="outlined"
        density="compact"
        readonly
      ></v-text-field>
    </div>
    <!-- Map creation dialog -->
    <v-dialog v-model="dialogMap" max-width="500">
      <template v-slot:activator="{ props: activatorProps }">
        <v-btn
          :disabled="loading"
          v-bind="activatorProps"
          class="bg-pink-darken-2"
          icon="mdi-target"
          variant="flat"
        ></v-btn>
      </template>
      <v-card :title="$t('labels.select-location')">
        <CreationMap :dialogMap="dialogMap"></CreationMap>
        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn :text="$t('labels.save')" @click="closeLocationDialog"></v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- END Map creation dialog -->
  </v-card>
  <!-- END Geotag container -->
  <v-row class="d-flex justify-end flex-row ma-1">
    <v-col cols="8" class="d-flex flex-column">
      <!-- Measure and KPI selection -->
      <v-card>
        <v-tabs v-model="tab" bg-color="transparent">
          <v-tab value="measures">{{ $t("labels.measures") }}</v-tab>
          <v-tab value="kpis">{{ $t("labels.kpis") }}</v-tab>
        </v-tabs>

        <v-tabs-window v-model="tab">
          <v-tabs-window-item value="measures">
            <div class="d-flex flex-column">
              <v-container>
                <v-row justify="start">
                  <v-col cols="12">
                    <v-text-field
                      ref="searchField"
                      v-model="search"
                      label="Search"
                      hide-details
                      single-line
                      density="compact"
                      clearable
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>

              <v-divider></v-divider>

              <v-virtual-scroll :items="searchResult" class="scroll">
                <template v-slot:default="{ item }">
                  <v-list-item v-if="!selected.includes(item)" :key="item.text" :disabled="loading">
                    <v-list-item-title v-text="item.text"></v-list-item-title>
                    <template v-slot:append
                      ><v-btn density="compact" @click="selected.push(item)"
                        >select</v-btn
                      ></template
                    >
                  </v-list-item>
                </template>
              </v-virtual-scroll>
            </div>
          </v-tabs-window-item>
          <v-tabs-window-item value="kpis">
            <div>
              <v-container>
                <v-row justify="start">
                  <v-col cols="12">
                    <v-text-field
                      ref="searchField"
                      v-model="search"
                      label="Search"
                      hide-details
                      single-line
                      density="compact"
                      clearable
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>

              <v-divider></v-divider>
              <v-virtual-scroll :items="searchResultKpis" class="scroll">
                <template v-slot:default="{ item }">
                  <v-list-item v-if="!selected.includes(item)" :key="item.text" :disabled="loading">
                    <v-list-item-title v-text="item.text"></v-list-item-title>
                    <template v-slot:append
                      ><v-btn density="compact" @click="selected.push(item)"
                        >select</v-btn
                      ></template
                    >
                  </v-list-item>
                </template>
              </v-virtual-scroll>

              <v-divider></v-divider>
            </div>
          </v-tabs-window-item>
        </v-tabs-window>
      </v-card>
    </v-col>
    <v-col cols="4" class="d-flex flex-column">
      <!-- END Measure and KPI selection -->

      <v-card class="flex-grow-1">
        <v-card-title class="bg-pink-darken-2"
          ><div height="10%">
            {{ $t("labels.selected") }}
          </div></v-card-title
        >

        <v-list
          ><v-list-item v-for="(selection, i) in selections" :key="selection.text">
            <v-list-item-title>{{ selection.text }}</v-list-item-title>
            <template v-slot:append
              ><v-btn :disabled="loading" @click="selected.splice(i, 1)">
                {{ $t("labels.delete") }}
              </v-btn></template
            >
          </v-list-item></v-list
        >
      </v-card>
    </v-col>
    <v-btn bg-color="purple" variant="text" class="me-auto">
      {{ $t("labels.cancel") }}
    </v-btn>
    <v-btn :loading="loading" bg-color="purple" variant="text">
      {{ $t("labels.back") }}
    </v-btn>
    <v-btn
      :disabled="!selected.length"
      :loading="loading"
      bg-color="purple"
      variant="text"
      @click="next"
    >
      {{ $t("labels.next") }}
    </v-btn>
  </v-row>
</template>

<script>
import CreationMap from "./components/CreationMap.vue";
import L from "leaflet";
import axios from "axios";
export default {
  name: "MapComponent",
  components: {
    CreationMap,
  },
  data: function () {
    return {
      tab: null,
      KPIs: [
        {
          text: "Table",
          icon: "mdi-nature",
        },
        {
          text: "Chair",
          icon: "mdi-glass-wine",
        },
        {
          text: "Keyobard",
          icon: "mdi-calendar-range",
        },
        {
          text: "Mouse",
          icon: "mdi-map-marker",
        },
        {
          text: "Window",
          icon: "mdi-bike",
        },
      ],
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

      marker: L.marker,
      markerLayer: L.layerGroup,

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

    closeLocationDialog() {
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
    //remove when attached to main project - get KPIs from parent
    getKPIsList() {
      this.loading = true;
      axios
        .get("https://indicators-manager-api.dev.dev-digital-enabler.eng.it/api/v1/kpi")
        .then((response) => {
          this.KPIs = response.data;
          console.log(this.KPIs);
          this.loading = false;
        })
        .catch((err) => {
          this.setError(err, false);
          this.loading = false;
        });
    },
    setError(err, prevent, actionEv) {
      console.log(err);
      if (!err || err.response.status >= 500) {
        err = {
          response: {
            status: "500",
            statusText: "server_error",
          },
        };
      }
      document.dispatchEvent(
        new CustomEvent("error-" + err.response.status, {
          detail: {
            preventDefault: prevent | false,
            callbackEvent: actionEv,
            data: err.response,
          },
        })
      );
    },
  },

  computed: {
    allItemsSelected() {
      let count = 0;
      this.items.forEach((item) => {
        if (this.selected.includes(item)) {
          count++;
        }
      });

      return this.items.length === count;
    },
    allKpisSelected() {
      let count = 0;
      this.KPIs.forEach((item) => {
        if (this.selected.includes(item)) {
          count++;
        }
      });

      return this.KPIs.length === count;
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
    searchResultKpis() {
      const search = this.search.toLowerCase();

      if (!search) return this.KPIs;

      return this.KPIs.filter((kpi) => {
        //convert kpi to text and looks in it for matches - might consider replacing for the actual measures
        const text = kpi.text.toLowerCase();

        return text.indexOf(search) > -1;
      });
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
.scroll {
  height: 30vh;
}
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
