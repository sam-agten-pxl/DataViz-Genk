<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  }
}
</script>

<style>
@font-face {
  font-family: "SansaPro";
  src: local("SansaPro"),
    url(./fonts/SansaPro-Normal.otf) format("truetype");
}

#app {
  font-family: "SansaPro";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
