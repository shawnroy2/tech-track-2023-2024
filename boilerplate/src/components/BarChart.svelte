<script>
    import { onMount, afterUpdate } from "svelte";
    import * as d3 from "d3";
  
    export let currentPlayer;
    export let data;
    export let selectedChart;
  
    const margin = { top: 20, right: 20, bottom: 50, left: 50 };
    const width = 500 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;
  
    let svg;
  
    onMount(() => {
      svg = d3.select("#chart").append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    });
  
    afterUpdate(() => {
      updateChart();
    });
  
    function updateChart() {
      if (selectedChart !== "barchart") return;
  
      const filterdataset = data.filter((speler) => speler.Player === currentPlayer);
      const maxTouchdowns = d3.max(filterdataset, (d) => d.AllTD);
  
      const xScale = d3.scaleLinear()
        .domain([0, maxTouchdowns])
        .range([0, width]);
  
      const yScale = d3.scaleBand()
        .domain(filterdataset.map((d) => d.Season.toString()))
        .range([0, height])
        .padding(0.4);
  
      const bars = svg.selectAll("rect")
        .data(filterdataset);
  
      bars.exit().remove();
  
      bars.enter().append("rect")
        .attr("x", 0)
        .attr("y", (d) => yScale(d.Season.toString()))
        .attr("width", (d) => Math.max((width / maxTouchdowns) * d.AllTD - 20, 20))
        .attr("height", 20)
        .attr("fill", "steelblue")
        .merge(bars)
        .transition()
        .duration(1000)
        .attr("x", 0)
        .attr("width", (d) => xScale(d.AllTD));
  
      // Voeg tekstlabels toe
      const labels = svg.selectAll(".label")
        .data(filterdataset);
  
      labels.exit().remove();
  
      labels.enter().append("text")
        .attr("class", "label")
        .text((d) => d.AllTD)
        .attr("x", (d) => xScale(d.AllTD) + 5)
        .attr("y", (d) => yScale(d.Season.toString()) + 20 / 2)
        .attr("text-anchor", "start")
        .attr("font-size", "12px")
        .attr("fill", "black")
        .merge(labels)
        .transition()
        .duration(1000)
        .text((d) => d.AllTD)
        .attr("x", (d) => xScale(d.AllTD) + 5);
    }
  </script>
  
  <style>
    /* Voeg eventuele stijlen hier toe */
    #chart {
      width: 300px;
      height: 300px;
      margin: 10px;
    }
  </style>
  
  <div id="chart"></div>
  