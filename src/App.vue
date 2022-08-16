<template>
  <div id="app">
    <!-- <MintPage/> -->
    <CandidatesList/>
  </div>
</template>

<script>
// import MintPage from './components/MintPage.vue'
import CandidatesList from './components/CandidatesList.vue';

export default {
  name: 'App',
  components: {
    // MintPage,
    CandidatesList
}
}
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
