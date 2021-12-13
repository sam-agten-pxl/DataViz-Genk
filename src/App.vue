<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
  </div>
</template>

<script>

import * as d3 from "d3";

export default {
  name: 'App',
  components: {
  },
  async mounted() {
    const data = await d3.csv('abonnees_genk.csv', this.typeConversion);
    const filteredData = this.filterData(data);
    const preparedData = this.prepareData(filteredData);
    console.log(preparedData);
  },
  methods: {
    typeConversion: function(d) {
      return {
        postcode: +d.Postcode,
        city: d.City
      }
    },
    filterData: function(d) {
      const filtered = d.filter(r=> {
        return r.postcode && r.postcode > 999 && r.postcode < 10000;
      });
      return filtered;
    },
    prepareData : function(data)
    {
      const dataMap = d3.rollup(
          data,
          r => d3.count(r, x => x.postcode),
          d => d.postcode
      )

      console.log(dataMap);

      const dataArray = Array.from(dataMap, d => ({ postcode: d[0], amount: d[1], city: data.find(e => e.postcode == d[0]).city.toUpperCase() }));
      const filteredDataArray = dataArray.filter(r => {
        return r.amount > 99;
      })
      return filteredDataArray;
    },
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
