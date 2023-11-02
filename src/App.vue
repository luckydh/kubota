<template>
  <div id="app">
    <main-app></main-app>
    <!-- <router-view /> -->
  </div>
</template>
<script>
import MainApp from "./views/MainApp.vue";
export default {
  components: { MainApp },
};
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.el-header {
  background-color: #d7dfe9;
  color: blue;
  text-align: center;
  line-height: 60px;
}
.active {
  background-color: #d7dfe9;
  color: blue;
}
</style>
