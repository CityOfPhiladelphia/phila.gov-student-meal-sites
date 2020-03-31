<template>
  <div>
    <h2>DISTRICT SITES</h2>
    <table>
      <thead>
        <tr>
          <th
            class="table-sort title"
            :class="districtSort"
            @click="toggleDist()"
          >
            <span>School</span>
          </th>
          <th>Address</th>
        </tr>
      </thead>
      <tbody>
      <tr v-for="site in filteredDistrict" 
          :key="site.OBJECTID">
          <td>{{site.SiteName }}</td>
         <td>{{ site.ADDRESS }} {{ site.ZipCode}}</td>
      </tr>
      </tbody>
    </table>
    <h2>CHARTER SITES</h2>
    <table>
      <thead>
        <tr>
          <th
            class="table-sort title"
            :class="charterSort"
            @click="toggleCharter()"
          >
            <span>School</span>
          </th>
          <th>Address</th>
        </tr>
      </thead>
      <tbody>
      <tr v-for="site in filteredCharter" 
          :key="site.OBJECTID">
          <td>{{site.SiteName }}</td>
         <td>{{ site.ADDRESS }} {{ site.ZipCode}}</td>
      </tr>
      </tbody>
    </table>
    <h2>PHA SITES</h2>
    <table>
      <thead>
        <tr>
          <th
            class="table-sort title"
            :class="PHASort"
            @click="togglePHA()"
          >
            <span>School</span>
          </th>
          <th>Address</th>
        </tr>
      </thead>
      <tbody>
      <tr v-for="site in filteredPHA" 
          :key="site.OBJECTID">
          <td>{{site.SiteName }}</td>
         <td>{{ site.ADDRESS }} {{ site.ZipCode}}</td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";

const endpoint = 
"https://services.arcgis.com/fLeGjb7u4uXqeF9q/arcgis/rest/services/Youth_Meal_Sites_3_20_2020_public_test/FeatureServer/0/query?where=1%3D1&outFields=*&f=pjson";

export default {
  name: 'StudentMeals',
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

      dEmpty: false,
      cEmpty: false,
      pEmpty: false,
    }
  },
  mounted() {
    this.fetchData()
  },
  methods: {
    fetchData: function() {
      axios.get(endpoint)
      .then((response) => {
        this.mealSites = response.data.features;

        response.data.features.forEach((item)=> {
          this.mealSites.push(item.attributes);
        })

        this.splitIntoArrays();
      })
      .catch((e) => {
        console.log(e)
      })
    },

    toggleDist() {
      this.districtSort = this.districtSort === 'asc' ? 'desc' : 'asc';
      this.filteredDistrict = this.sortSites(this.filteredDistrict, this.districtSort)
    },

    toggleCharter() {
      this.charterSort = this.charterSort === 'asc' ? 'desc' : 'asc';
      this.filteredCharter = this.sortSites(this.filteredCharter, this.charterSort)
    },

    togglePHA() {
      this.PHASort = this.PHASort === 'asc' ? 'desc' : 'asc';
      this.filteredPHA = this.sortSites(this.filteredPHA, this.PHASort)
    },

    sortSites:function(toSort, mod) {
      return toSort.sort((a, b) => {
         let modifier = -1;
          if(mod === 'desc') {
            modifier = 1;
          }
        var textA = a.SiteName.toUpperCase();
        var textB = b.SiteName.toUpperCase();
          return (textA < textB) ? ( -1 * modifier) : (textA > textB) ? (1 * modifier) : 0;
      });
    },

    splitIntoArrays: function () {
      let districtArr = this.mealSites.filter(site => site.Status === "District");
      this.districtSites = districtArr;
      this.filteredDistrict = this.sortSites(districtArr, this.districtSort);

      let charterArr = this.mealSites.filter(site => site.Status === "Charter");
      this.charterSites = charterArr;
      this.filteredCharter = this.sortSites(charterArr, this.charterSort);

      let PHAArr = this.mealSites.filter(site => site.Status === "PHA");
      this.PHASites = PHAArr;
      this.filteredPHA = this.sortSites(PHAArr, this.PHASort);
    }
  },
}
</script>

<style scoped>

</style>
