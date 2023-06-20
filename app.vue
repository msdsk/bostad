<template>
  <div>
    <h1>Housing stock demand vs prices</h1>
    <svg ref="svgRef" id="result" :width="width" :height="height">
      <g class="grid" ref="xGridRef" :transform="`translate(0, ${height - margin})`">
      </g>
      <g class="grid" ref="yGridRef" :transform="`translate(${margin},0)`">
      </g>
      <path class="graph prices" :d="line(priceWInflation)"></path>
      <path class="graph housing" :d="line(capitaPerHouse)"></path>
      <g class="axis" ref="xAxisRef" :transform="`translate(0, ${height - margin})`">
      </g>
      <g class="axis" ref="yAxisRef" :transform="`translate(${margin},0)`">
      </g>
    </svg>
  </div>
</template>

<script setup lang="ts">
import inflation from '~/assets/data/inflation.json'
import prices from "~/assets/data/prices.json"
import population from "~/assets/data/population.json"
import housingUnits from "~/assets/data/housing_units.json"

const capitaPerHouse = population.map(({ year, value }, i) => ({ year: year, value: value / housingUnits[i].value }))
const priceWInflation = prices.map(({ year, value }, i) => ({ year: year, value: value / inflation[i].value }))

type DataPoint = {
  year: number;
  value: number;
}
type LineData = DataPoint[]

import * as d3 from "d3"

const xAxisRef = ref()
const yAxisRef = ref()
const xGridRef = ref()
const yGridRef = ref()

const width = 1200
const height = 800
const margin = 50

const startDate = 2010
const EndDate = 2022

// const normalize = (arr: LineData) => {
//   return arr.map(({ value }) => value / arr[0].value * 100)
// }

// const normalizedData = ([] as number[]).concat(...[prices, capitaPerHouse].map(normalize))
// const min = Math.min(...normalizedData)
// const max = Math.max(...normalizedData)
const min = 100
const max = 170

const x = d3.scaleLinear([startDate, EndDate], [margin, width - margin])
const y = d3.scaleLinear([max, min], [margin, height - margin])

console.log(x(2013), x(2022))
console.log(y(100), y(200))

onMounted(() => {
  //grid
  d3.select(xGridRef.value).call(d3.axisBottom(x).tickSize(-height + margin * 2).tickFormat(() => ""))
  d3.select(yGridRef.value).call(d3.axisLeft(y).tickSize(-width + margin * 2).tickFormat(() => ""))

  //axes
  d3.select(xAxisRef.value).call(d3.axisBottom(x).tickSize(15).tickFormat(year => `${year}`))
  d3.select(yAxisRef.value).call(d3.axisLeft(y).tickSize(15))
})

const line = (data: LineData) => d3.line()(data.map(point => ([x(point.year), y(point.value / data[0].value * 100)]))) || undefined
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700&display=swap');

html {
  font-size: 22px;
  font-family: "Open Sans";
}

h1 {
  font-size: 2rem;
}

path {
  stroke: #000;
  fill: none;
}

.graph {
  stroke-width: 4px;
  stroke-linecap: round;
  stroke-linejoin: round;
}

text {
  font-size: 20px;
}

.grid {
  opacity: .15;
}

.prices {
  stroke: rgb(224, 101, 0);
}

.housing {
  stroke: rgb(140, 0, 255);
}
</style>