<template>
  <div>
    <div id="line-wrapper"></div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import * as d3 from 'd3'

export default Vue.extend({
  name: 'LineChart',
  data() {
    return {
      svg: null,
      width: null,
      height: null,
      margin: null,
    }
  },
  mounted() {
    this.margin = {top: 50, right: 50, bottom: 50, left: 50}
    this.width = this.$el.clientWidth - this.margin.left - this.margin.right
    this.height = 500 - this.margin.top - this.margin.bottom

    this.svg = d3.select('#line-wrapper')
      .append('svg')
      .attr('width', this.width + this.margin.left + this.margin.right)
      .attr('height', this.height + this.margin.top + this.margin.bottom)
      .append("g")
      .attr("transform", "translate(" + this.margin.left + "," + this.margin.top + ")")

//Read the data
    d3.csv("/csv/-1.5-1.100.npy.csv", (d) => {
      return { timestep : parseInt(d.timestep), value : parseFloat(d.value) }
    }).then(this.updatePlot)
  },
  methods: {
    updatePlot(data) {


      // Add X axis --> it is a date format
      var x = d3.scaleLinear()
        .domain(d3.extent(data, function (d) {
          return parseInt(d.timestep);
        }))
        .range([0, this.width]);
      var xAxis = this.svg.append("g")
        .attr("transform", "translate(0," + this.height + ")")
        .call(d3.axisBottom(x));

      // Add Y axis
      var y = d3.scaleLinear()
        .domain([d3.min(data, function (d) {
          return parseFloat(d.value);
        }), d3.max(data, function (d) {
          return parseFloat(d.value);
        })])
        .range([this.height, 0]);
      var yAxis = this.svg.append("g")
        .call(d3.axisLeft(y));

      // Add a clipPath: everything out of this area won't be drawn.
      var clip = this.svg.append("defs").append("svg:clipPath")
        .attr("id", "clip")
        .append("svg:rect")
        .attr("width", this.width)
        .attr("height",this.height)
        .attr("x", 0)
        .attr("y", 0);

      // Add brushing
      var brush = d3.brushX()                   // Add the brush feature using the d3.brush function
        .extent([[0, 0], [this.width, this.height]])  // initialise the brush area: start at 0,0 and finishes at width,height: it means I select the whole graph area
        .on("end", updateChart)               // Each time the brush selection changes, trigger the 'updateChart' function

      // Create the line variable: where both the line and the brush take place
      var line = this.svg.append('g')
        .attr("clip-path", "url(#clip)")

      // Add the line
      line.append("path")
        .datum(data)
        .attr("class", "line")  // I add the class line to be able to modify this line later on.
        .attr("fill", "none")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 1.5)
        .attr("d", d3.line()
          .x(function (d) {
            return x(parseInt(d.timestep))
          })
          .y(function (d) {
            return y(parseFloat(d.value))
          })
        )

      // Add the brushing
      line
        .append("g")
        .attr("class", "brush")
        .call(brush);

      // A function that set idleTimeOut to null
      var idleTimeout

      function idled() {
        idleTimeout = null;
      }

      // A function that update the chart for given boundaries
      function updateChart(event,d) {

        // What are the selected boundaries?
        var extent = event.selection

        // If no selection, back to initial coordinate. Otherwise, update X axis domain
        if(!extent){
          if (!idleTimeout) return idleTimeout = setTimeout(idled, 350); // This allows to wait a little bit
          x.domain([ 4,8])
        }else{
          x.domain([ x.invert(extent[0]), x.invert(extent[1]) ])
          line.select(".brush").call(brush.move, null) // This remove the grey brush area as soon as the selection has been done
        }

        // Update axis and line position
        xAxis.transition().duration(1000).call(d3.axisBottom(x))
        line
          .select('.line')
          .transition()
          .duration(1000)
          .attr("d", d3.line()
            .x(function(d) { return x(d.timestep) })
            .y(function(d) { return y(parseFloat(d.value)) })
          )
      }

      // If user double click, reinitialize the chart
      this.svg.on("dblclick", function () {
        x.domain(d3.extent(data, function (d) {
          return parseInt(d.timestep);
        }))
        xAxis.transition().call(d3.axisBottom(x))
        line
          .select('.line')
          .transition()
          .attr("d", d3.line()
            .x(function (d) {
              return x(parseInt(d.timestep))
            })
            .y(function (d) {
              return y(parseFloat(d.value))
            })
          )
      });
    }
  }
})
</script>
