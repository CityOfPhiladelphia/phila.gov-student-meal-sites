<template>
  <div>
    <div v-show="loading" class="mtm center">
      <i class="fas fa-spinner fa-spin fa-3x" />
    </div>
    <div v-show="failure" class="h3 mtm center">Sorry, there was a problem. Please try again.</div>
    <div v-show="!loading && !failure">
      <h5>
        <b>District schools open Mondays and Thursdays, 9 a.m. – noon.</b>
      </h5>
      <p>Families will receive six meals per student—3 breakfasts and 3 lunches</p>
      <div class="search">
        <input
          id="district-search-bar"
          v-model="districtSearch"
          class="search-field"
          type="text"
          placeholder="Filter by site name or address"
        />
        <input ref="district-search-bar" type="submit" class="search-submit" value="Search" />
        <button
          v-if="districtSearch.length > 0"
          class="clear-search-btn"
          @click="clearDisctrictSearch()"
        >
          <i class="fas fa-times" />
        </button>
      </div>
      <div v-show="!loading && districtEmpty" class="h3 mtm center">Sorry, there are no results.</div>
      <table v-show="!districtEmpty && !loading && !failure">
        <thead>
          <tr>
            <th class="table-sort title" :class="districtSort" @click="toggleDist()">
              <span>School</span>
            </th>
            <th>Address (ZIP code)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="site in filteredDistrict" :key="site.OBJECTID">
            <td>{{site.SiteName }}</td>
            <td>{{ site.ADDRESS }} ({{ site.ZipCode}})</td>
          </tr>
        </tbody>
      </table>
      <h5>
        <b>Charter School locations open on varying days from 9 a.m. to noon. (contact schools or visit their websites for exact days)</b>
      </h5>
      <div class="search">
        <input
          id="charter-search-bar"
          v-model="charterSearch"
          class="search-field"
          type="text"
          placeholder="Filter by site name or address"
        />
        <input ref="charter-search-bar" type="submit" class="search-submit" value="Search" />
        <button
          v-if="charterSearch.length > 0"
          class="clear-search-btn"
          @click="clearCharterSearch()"
        >
          <i class="fas fa-times" />
        </button>
      </div>
      <div v-show="!loading && charterEmpty" class="h3 mtm center">Sorry, there are no results.</div>
      <table v-show="!charterEmpty && !loading && !failure">
        <thead>
          <tr>
            <th class="table-sort title" :class="charterSort" @click="toggleCharter()">
              <span>School</span>
            </th>
            <th>Address (ZIP code)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="site in filteredCharter" :key="site.OBJECTID">
            <td>{{ site.SiteName }}</td>
            <td>{{ site.ADDRESS }} ({{ site.ZipCode}})</td>
          </tr>
        </tbody>
      </table>
      <h5>
        <b>The Philadelphia Housing Authority has six community centers open for “grab-and-go” meals for breakfast and lunch from Monday through Friday, 9 a.m. to noon.</b>
      </h5>
      <div v-show="!loading && PHAEmpty" class="h3 mtm center">Sorry, there are no results.</div>
      <table v-show="!PHAEmpty && !loading && !failure">
        <thead>
          <tr>
            <th class="table-sort title" :class="PHASort" @click="togglePHA()">
              <span>PHA community center</span>
            </th>
            <th>Address (ZIP code)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="site in filteredPHA" :key="site.OBJECTID">
            <td>{{ site.SiteName }}</td>
            <td>{{ site.ADDRESS }} ({{ site.ZipCode}})</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import VueFuse from "vue-fuse";
import Vue from "vue";

Vue.use(VueFuse);

const endpoint =
  "https://services.arcgis.com/fLeGjb7u4uXqeF9q/arcgis/rest/services/Youth_Meal_Sites_3_20_2020_public_test/FeatureServer/0/query?where=1%3D1&outFields=*&f=pjson";

export default {
  name: "StudentMeals",
  data: function() {
    return {
      mealSites: [],
      districtSites: [],
      charterSites: [],
      PHASites: [],
      filteredDistrict: [],
      filteredCharter: [],
      filteredPHA: [],
      districtSort: "desc",
      charterSort: "desc",
      PHASort: "desc",

      loading: false,
      failure: false,

      districtEmpty: false,
      charterEmpty: false,
      PHAEmpty: false,

      districtSearch: "",
      charterSearch: "",
      PHASearch: "",

      searchOptions: {
        shouldSort: true,
        threshold: 0.1,
        tokenize: true,
        keys: ["SiteName", "ADDRESS", "ZipCode"]
      }
    };
  },
  watch: {
    districtSearch(val) {
      this.filteredDistrict = this.searchTable(
        this.districtSites,
        "district",
        val
      );
    },

    charterSearch(val) {
      this.filteredCharter = this.searchTable(
        this.charterSites,
        "charter",
        val
      );
    },

    PHASearch(val) {
      this.filteredPHA = this.searchTable(this.PHASites, "PHA", val);
    },

    filteredDistrict() {
      if (!this.failure) {
        if (this.filteredDistrict.length == 0) {
          this.districtEmpty = true;
        } else {
          this.districtEmpty = false;
        }
      }
    },

    filteredCharter() {
      if (!this.failure) {
        if (this.filteredCharter.length == 0) {
          this.charterEmpty = true;
        } else {
          this.charterEmpty = false;
        }
      }
    },

    filteredPHA() {
      if (!this.failure) {
        if (this.filteredPHA.length == 0) {
          this.PHAEmpty = true;
        } else {
          this.PHAEmpty = false;
        }
      }
    }
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    fetchData: function() {
      this.loading = true;
      axios
        .get(endpoint)
        .then(response => {
          this.mealSites = response.data.features;

          response.data.features.forEach(item => {
            this.mealSites.push(item.attributes);
          });
          this.failure = false;
          this.loading = false;

          this.splitIntoArrays();
        })
        .catch(e => {
          console.log(e);
          this.loading = false;
          this.failure = true;
        });
    },

    toggleDist() {
      this.districtSort = this.districtSort === "asc" ? "desc" : "asc";
      this.filteredDistrict = this.sortSites(
        this.filteredDistrict,
        this.districtSort
      );
    },

    toggleCharter() {
      this.charterSort = this.charterSort === "asc" ? "desc" : "asc";
      this.filteredCharter = this.sortSites(
        this.filteredCharter,
        this.charterSort
      );
    },

    togglePHA() {
      this.PHASort = this.PHASort === "asc" ? "desc" : "asc";
      this.filteredPHA = this.sortSites(this.filteredPHA, this.PHASort);
    },

    sortSites: function(toSort, mod) {
      return toSort.sort((a, b) => {
        let modifier = -1;
        if (mod === "desc") {
          modifier = 1;
        }
        var textA = a.SiteName.toUpperCase();
        var textB = b.SiteName.toUpperCase();
        return textA < textB ? -1 * modifier : textA > textB ? 1 * modifier : 0;
      });
    },

    splitIntoArrays: function() {
      let districtArr = this.mealSites.filter(
        site => site.Status === "District"
      );
      this.districtSites = this.sortSites(districtArr, this.districtSort);
      this.filteredDistrict = this.districtSites;

      let charterArr = this.mealSites.filter(site => site.Status === "Charter");
      this.charterSites = this.sortSites(charterArr, this.charterSort);
      this.filteredCharter = this.charterSites;

      let PHAArr = this.mealSites.filter(site => site.Status === "PHA");
      this.PHASites = this.sortSites(PHAArr, this.PHASort);
      this.filteredPHA = this.PHASites;
    },

    searchTable: function(siteArr, whichArr, search) {
      if (search) {
        this.$search(search, siteArr, this.searchOptions).then(sites => {
          if (whichArr === "district") {
            this.filteredDistrict = sites;
          } else if (whichArr === "charter") {
            this.filteredCharter = sites;
          } else if (whichArr === "PHA") {
            this.filteredPHA = sites;
          }
        });
      } else {
        return siteArr;
      }
    },

    clearDisctrictSearch: function() {
      this.districtSearch = "";
    },

    clearCharterSearch: function() {
      this.charterSearch = "";
    },

    clearPHASearch: function() {
      this.PHASearch = "";
    }
  }
};
</script>

<style scoped>
.clear-search-btn {
  position: absolute;
  top: 16px;
  right: 70px;
  padding: 0;
  font-size: 20px;
  background-color: #fff;
  opacity: 0.8;
  z-index: 999;
  cursor: pointer;
  color: rgba(60, 60, 60, 0.5);
}
</style>
