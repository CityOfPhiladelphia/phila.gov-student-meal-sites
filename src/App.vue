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
thead tr td {
    width: 50%;
  }
</style>
