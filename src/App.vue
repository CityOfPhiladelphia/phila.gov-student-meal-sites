<template>
  <div id="vue-app">
    <StudentMeals />
  </div>
</template>

<script>
import StudentMeals from './components/StudentMeals.vue'


export default {
  name: 'App',
  components: {
    StudentMeals,
  }
}
</script>

<style>
/* #vue-app {
  width: 70rem;
  margin: 0 auto;
} */

thead tr th {
    width: 50%;
  }
</style>
