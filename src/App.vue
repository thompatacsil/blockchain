<template>
  <div id="app">
    <!-- <MintPage/> -->
    <CandidatesList/>
  </div>
</template>

<script>
// import MintPage from './components/MintPage.vue'
import Vue from 'vue';
import CandidatesList from './components/CandidatesList.vue';
import VueSweetAlert2 from 'vue-sweetalert2';
import 'sweetalert2/dist/sweetalert2.min.css';

export default {
  name: 'App',
  components: {
    // MintPage,
    CandidatesList
  }
}

Vue.use(VueSweetAlert2);
</script>

<style>
*{
  box-sizing: border-box;
}

body, html{
  height: 100%!important;
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  overflow-x: hidden;
  height: 100vh!important;
}
</style>
