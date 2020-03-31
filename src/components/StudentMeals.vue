<template>
  <div>
    <h2>DISTRICT SITES</h2>
    <table>
      <thead>
        <tr>
          <td>Name</td>
          <td>Address</td>
        </tr>
      </thead>
      <tbody>
      <tr v-for="site in districtSites" 
          :key="site.attributes.OBJECTID">
          <td>{{site.attributes.SiteName }}</td>
         <td>{{ site.attributes.ADDRESS }} {{ site.attributes.ZipCode}}</td>
      </tr>
      </tbody>
    </table>
    <h2>CHARTER SITES</h2>
    <table>
      <thead>
        <tr>
          <td>Name</td>
          <td>Address</td>
        </tr>
      </thead>
      <tbody>
      <tr v-for="site in charterSites" 
          :key="site.attributes.OBJECTID">
          <td>{{site.attributes.SiteName }}</td>
         <td>{{ site.attributes.ADDRESS }} {{ site.attributes.ZipCode}}</td>
      </tr>
      </tbody>
    </table>
    <h2>PHA SITES</h2>
    <table>
      <thead>
        <tr>
          <td>Name</td>
          <td>Address</td>
        </tr>
      </thead>
      <tbody>
      <tr v-for="site in PHASites" 
          :key="site.attributes.OBJECTID">
          <td>{{site.attributes.SiteName }}</td>
         <td>{{ site.attributes.ADDRESS }} {{ site.attributes.ZipCode}}</td>
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
        this.splitIntoArrays();
      })
      .catch((e) => {
        console.log(e)
      })
    },

    splitIntoArrays: function () {
      let districtArr = this.mealSites.filter(site => site.attributes.Status === "District");
      this.districtSites = districtArr;

      let charterArr = this.mealSites.filter(site => site.attributes.Status === "Charter");
      this.charterSites = charterArr;

      let PHAArr = this.mealSites.filter(site => site.attributes.Status === "PHA");
      this.PHASites = PHAArr;
    }
  },
}
</script>

<style scoped>

</style>
