<template>
  <div id="app">
    <File />
  </div>
</template>

<script>
import File from "./components/File.vue";

export default {
  name: "App",
  components: {
    File
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
