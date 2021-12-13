<template>
  <div id="app">
    <div class="titlebox">
      <img class="logo" alt="Genk Logo" src="./assets/logo.png" width="150"/>
      <div id="title">
        <h1>RACING GENK</h1>
        <p>Abonnees per gemeente met minstens <b>100</b> abonnees</p>
      </div>
    </div>
      <div class="filter-buttons">
      <input type="radio" id="town" value="town" v-model="filter" />
      <label for="town">Gemeente</label>
      <input type="radio" id="postalCode" value="postalCode" v-model="filter" />
      <label for="postalCode">Postcode</label>
    </div>
    <div id="content">
      <svg id="canvas" :width=width height=1000>
          <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle" v-on:mouseover="selectedData = d">
            <circle :r="calculateRadius(d)" />
            <circle class="circle-overlay" :r="calculateRadius(d)" />
            <text y=75 class="label">
              {{filter == 'town' ? d.city : d.postcode.toString()}}
            </text>
          </g>
      </svg>

      <div v-for="(d, index) in data" :key="d.postcode">
          <Card :show="selectedData == d" :x="calculateXPosition(index)" :y="calculateYPosition(index)" :title="filter == 'town' ? d.city : d.postcode.toString()" :content="d.amount.toString()"/>
      </div>
    </div>
  </div>
</template>

<script>

import * as d3 from "d3";
import Card from './components/Card.vue';

export default {
  name: 'App',
  components: {
    Card
  },
  data: function() {
    return {
      width: 1800,
      data: Object,
      rowLength: 9,
      selectedData: Object,
      filter: 'town'
    }
  },
  async mounted() {
    const data = await d3.csv('abonnees_genk.csv', this.typeConversion);
    const filteredData = this.filterData(data);
    const preparedData = this.prepareData(filteredData);
    console.log(preparedData);

    this.data = preparedData;
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
    calculateXPosition: function(i)
    {
      const w = this.width - 75 * 2;
      const div = w / this.rowLength;
      return 75 + div/2 + (i % this.rowLength) * div;
    },
    calculateYPosition: function(i)
    {
      return 50+ Math.floor(i/ this.rowLength) * 150;
    },
    calculateRadius: function(da)
    {
      const extents = d3.extent(this.data, d => d.amount);
      const t = da.amount / extents[1];
      return t * 50;
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

.data-circle {
  fill: #1a7ebe;
}

.label {
  fill: #1a7ebe;
  text-anchor: middle;
}

.circle-overlay {
  fill: #024d9d;
  transform: scale(0);
  transition: ease-out 300ms;
}

.data-circle:hover .circle-overlay {
  transform: scale(1);
}


.titlebox {
  display: flex;
  height: 200px;
  margin-left: 150px;
}

#title h1 {
  margin-bottom: 0px;
}

#title p {
  margin-top : 0px;
}

#title {
  margin-left: 50px;
  padding-top: 50px;
}

h1 {
  text-align: left;
}

.filter-buttons {
  margin-top: 50px;
  margin-bottom: 20px;
  margin-right: 150px;
  display:flex;
  justify-content: right;
}


</style>
