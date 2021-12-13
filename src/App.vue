<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
  },
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

#content {
  position: relative;
}

img {
  display: block;
  width: auto;
  height: auto;
}

*::before,
*::after {
  box-sizing: border-box;
}

body {

  --dark-blue-color: #024d9d;
  --light-blue-color: #1a7ebe;

  display: grid;
  min-height: 100vh;
  line-height: 1.6;
  background: white;
  font-family: sans-serif;
  justify-content: center;
}

</style>
