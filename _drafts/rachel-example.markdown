---
title: Rachel's example
---

<script src="https://d3js.org/d3.v4.min.js"></script>

<style>
  path {
    fill: none;
    stroke: black;
  }
</style>
<div id="chart"></div>
<script>

var width = 800,
  height = 400;

var stddev = 1;

var dataset = d3.range(-10, 10, 0.05).map(function(t) {
  return {"x":t, "y":Math.exp(-Math.pow(t,2)/(2*Math.pow(stddev,2)))};
});

var svg = d3.select("#chart").append("svg")
  .attr("width",width).attr("height",height);

var xScale = d3.scaleLinear()
  .domain([-10,10]).range([40, width-20]),
    yScale = d3.scaleLinear()
  .domain([0, 2]).range([height-40, 20]);

// fill in dataMin and dataMax with y axis extent
// the 40 & 20 numbers are padding for axes below

var line = d3.line()
  .x(function(d) { return xScale(d.x); })
  .y(function(d) { return yScale(d.y); });

// above is assuming datapoints are in the format { x : , y : }

svg.append("path").datum(dataset).attr("d",line);
// dataset being your array of x,y points

var circles = svg.selectAll("circle").data(dataset);
circles.enter().append("circle").attr("r",3).attr("fill","#555")
  .attr("cx",function(d){ return xScale(d.x); })
  .attr("cy",function(d){ return yScale(d.y); });

// axes

var xAxis = d3.axisBottom(xScale.nice());

svg.append("g").attr("class","axis")
  .attr("transform","translate(0,"+(height-40)+")")
  .call(xAxis);

var yAxis = d3.axisLeft(yScale.nice());

svg.append("g").attr("class","axis")
  .attr("transform","translate(5,20)")
  .call(yAxis);
</script>


<div id="chart2"></div>
<style>
  path {
    fill: none;
    stroke: black;
  }
</style>
<script>
var width = 800,
  height = 400;

var stddev = 1;
var maxVal = 1/Math.sqrt(2*Math.PI*Math.pow(stddev,2))*Math.exp(-Math.pow(0,2)/(2*Math.pow(stddev,2)));

var dataset = d3.range(-10, 10, 0.01).map(function(t) {
  return {"x":t, "y": 1/Math.sqrt(2*Math.PI*Math.pow(stddev,2))*Math.exp(-Math.pow(t,2)/(2*Math.pow(stddev,2)))};
});

var svg = d3.select("#chart2").append("svg")
  .attr("width",width).attr("height",height);

var xScale = d3.scaleLinear()
  .domain([-10,10]).range([40, width-20]),
    yScale = d3.scaleLinear()
  .domain([0, maxVal]).range([height-40, 20]);

// fill in dataMin and dataMax with y axis extent
// the 40 & 20 numbers are padding for axes below

var line = d3.line()
  .x(function(d) { return xScale(d.x); })
  .y(function(d) { return yScale(d.y); })
  .curve(d3.curveCardinal);

// above is assuming datapoints are in the format { x : , y : }

svg.append("path").datum(dataset).attr("d",line);
// dataset being your array of x,y points

// var circles = svg.selectAll("circle").data(dataset);
// circles.enter().append("circle").attr("r",3).attr("fill","#555")
//   .attr("cx",function(d){ return xScale(d.x); })
//   .attr("cy",function(d){ return yScale(d.y); });

// axes

var xAxis = d3.axisBottom(xScale.nice());

svg.append("g").attr("class","axis")
  .attr("transform","translate(0,"+(height-40)+")")
  .call(xAxis);

var yAxis = d3.axisLeft(yScale.nice());

svg.append("g").attr("class","axis")
  .attr("transform","translate(40,0)")
  .call(yAxis);

</script>
