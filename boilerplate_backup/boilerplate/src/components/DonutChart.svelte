<script>
    import { onMount, afterUpdate } from "svelte";
    import * as d3 from "d3";
  
    export let currentPlayer;
    export let data;
  
    const width = 300;
    const height = 300;
    const margin = { top: 20, right: 20, bottom: 20, left: 20 };
    const innerWidth = width - margin.left - margin.right;
    const innerHeight = height - margin.top - margin.bottom;
    const radius = Math.min(innerWidth, innerHeight) / 2;
  
    let svgDonut;
  
    onMount(() => {
      svgDonut = d3.select("#donut-chart").append("svg")
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", `translate(${width / 2},${height / 2})`);
    });
  
    afterUpdate(() => {
      createDonutChart();
    });
  
    function createDonutChart() {
      // Haal de gefilterde dataset op voor de huidige speler
      const filterdataset = data.filter((speler) => speler.Player === currentPlayer);
  
      // Data voor de donutchart
      const donutChartData = [filterdataset[0].AllTD, filterdataset[0].G];
  
      // Creëer de donutchart
      const color = d3.scaleOrdinal().range(["#66c2a5", "#fc8d62"]);
      const pie = d3.pie();
      const arc = d3.arc().innerRadius(radius - 40).outerRadius(radius);
  
      svgDonut.selectAll("*").remove(); // Verwijder oude elementen voordat je de nieuwe tekent
  
      svgDonut.selectAll("path")
        .data(pie(donutChartData))
        .enter().append("path")
        .attr("d", arc)
        .attr("fill", (d, i) => color(i))
        .append("title")
        .text((d) => d.data);
  
      svgDonut.append("text")
        .attr("text-anchor", "middle")
        .attr("font-size", "16px")
        .attr("dy", "0.35em")
        .text("Chart Title");
    }
  </script>
  
  <style>
    /* Voeg eventuele stijlen hier toe */
    #donut-chart {
      width: 300px;
      height: 300px;
      margin: 10px;
    }
  </style>
  
  <div id="donut-chart"></div>
  