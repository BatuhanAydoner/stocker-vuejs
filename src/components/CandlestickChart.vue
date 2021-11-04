<template>
  <svg id="container"></svg>
</template>

<script>
import * as d3 from "d3";
export default {
  props: {
    stockDetails: {
      type: Array,
      required: true,
    },
  },
  watch: {
    stockDetails(val) {
      if (val.length > 0) {
        this.createCandlestickChart();
      }
    },
  },
  methods: {
    createCandlestickChart() {
      const history = this.stockDetails;
      const wrap = this.wrap;
      d3.csv("").then(function (prices) {
        prices = history;
        console.log(prices);
        /* const months = {
      0: "Jan",
      1: "Feb",
      2: "Mar",
      3: "Apr",
      4: "May",
      5: "Jun",
      6: "Jul",
      7: "Aug",
      8: "Sep",
      9: "Oct",
      10: "Nov",
      11: "Dec",
    }; */

        var dateFormat = d3.timeParse("%Y-%m-%d");
        for (var i = 0; i < prices.length; i++) {
          prices[i]["timestamp"] = dateFormat(prices[i]["timestamp"]);
        }

        const margin = { top: 15, right: 65, bottom: 205, left: 50 },
          w = 1000 - margin.left - margin.right,
          h = 625 - margin.top - margin.bottom;

        var svg = d3
          .select("#container")
          .attr("width", w + margin.left + margin.right)
          .attr("height", h + margin.top + margin.bottom)
          .append("g")
          .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

        let dates = prices.map((price) => {
          return price["timestamp"];
        });

        //var xmin = d3.min(prices.map((r) => r.Date.getTime()));
        //var xmax = d3.max(prices.map((r) => r.Date.getTime()));
        var xScale = d3.scaleLinear().domain([-1, dates.length]).range([0, w]);
        //var xDateScale = d3.scaleQuantize().domain([0, dates.length]).range(dates);
        let xBand = d3
          .scaleBand()
          .domain(d3.range(-1, dates.length))
          .range([0, w])
          .padding(0.3);
        var xAxis = d3.axisBottom().scale(xScale);
        /* .tickFormat(function (d) {
        d = dates[d];
        hours = d.getHours();
        minutes = (d.getMinutes() < 10 ? "0" : "") + d.getMinutes();
        amPM = hours < 13 ? "am" : "pm";
        return (
          hours +
          ":" +
          minutes +
          amPM +
          " " +
          d.getDate() +
          " " +
          months[d.getMonth()] +
          " " +
          d.getFullYear()
        );
      }); */

        svg
          .append("rect")
          .attr("id", "rect")
          .attr("width", w)
          .attr("height", h)
          .style("fill", "none")
          .style("pointer-events", "all")
          .attr("clip-path", "url(#clip)");

        var gX = svg
          .append("g")
          .attr("class", "axis x-axis") //Assign "axis" class
          .attr("transform", "translate(0," + h + ")")
          .call(xAxis);

        gX.selectAll(".tick text").call(wrap, xBand.bandwidth());

        var ymin = d3.min(prices.map((r) => r.low));
        var ymax = d3.max(prices.map((r) => r.high));
        var yScale = d3.scaleLinear().domain([ymin, ymax]).range([h, 0]).nice();
        var yAxis = d3.axisLeft().scale(yScale);

        svg.append("g").attr("class", "axis y-axis").call(yAxis);

        var chartBody = svg
          .append("g")
          .attr("class", "chartBody")
          .attr("clip-path", "url(#clip)");

        // draw rectangles
        chartBody
          .selectAll(".candle")
          .data(prices)
          .enter()
          .append("rect")
          .attr("x", (d, i) => xScale(i) - xBand.bandwidth())
          .attr("class", "candle")
          .attr("y", (d) => yScale(Math.max(d.open, d.close)))
          .attr("width", xBand.bandwidth())
          .attr("height", (d) =>
            d.open === d.close
              ? 1
              : yScale(Math.min(d.open, d.close)) - yScale(Math.max(d.open, d.close))
          )
          .attr("fill", (d) =>
            d.open === d.close ? "silver" : d.open > d.close ? "red" : "green"
          );

        // draw high and low
        chartBody
          .selectAll("g.line")
          .data(prices)
          .enter()
          .append("line")
          .attr("class", "stem")
          .attr("x1", (d, i) => xScale(i) - xBand.bandwidth() / 2)
          .attr("x2", (d, i) => xScale(i) - xBand.bandwidth() / 2)
          .attr("y1", (d) => yScale(d.high))
          .attr("y2", (d) => yScale(d.low))
          .attr("stroke", (d) =>
            d.open === d.close ? "white" : d.open > d.close ? "red" : "green"
          );

        svg
          .append("defs")
          .append("clipPath")
          .attr("id", "clip")
          .append("rect")
          .attr("width", w)
          .attr("height", h);
      });
    },
    wrap(text, width) {
      text.each(function () {
        var text = d3.select(this),
          words = text.text().split(/\s+/).reverse(),
          word,
          line = [],
          lineNumber = 0,
          lineHeight = 1.1, // ems
          y = text.attr("y"),
          dy = parseFloat(text.attr("dy")),
          tspan = text
            .text(null)
            .append("tspan")
            .attr("x", 0)
            .attr("y", y)
            .attr("dy", dy + "em");
        while ((word = words.pop())) {
          line.push(word);
          tspan.text(line.join(" "));
          if (tspan.node().getComputedTextLength() > width) {
            line.pop();
            tspan.text(line.join(" "));
            line = [word];
            tspan = text
              .append("tspan")
              .attr("x", 0)
              .attr("y", y)
              .attr("dy", ++lineNumber * lineHeight + dy + "em")
              .text(word);
          }
        }
      });
    },
  },
};
</script>