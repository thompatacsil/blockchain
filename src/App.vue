<template>
  <div id="app">
    <CandidatesList/>
  </div>
</template>

<script>
import CandidatesList from './components/CandidatesList.vue'

export default {
  name: 'App',
  components: {
    CandidatesList,
  }
}
</script>

<style>
*{
  box-sizing: border-box;
}

body, html{
  height: 100%!important;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
  overflow-x: hidden;
  height: 100vh!important;
}

.div-container{
  background-color: #eee;
  margin-bottom: 10px;
  margin-right: 10px;
  padding: 10px;
  border-radius: 10px;
  box-shadow: #9a9a9a 2px 2px 3px;
}
</style>
