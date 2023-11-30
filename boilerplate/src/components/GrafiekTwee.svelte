<!-- DonutChart.svelte -->
<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
  
    let selectedSeason = 2018;
    let selectedStat = "G";
    let dataset = {
      "G": 16,
      "RshTD": 11,
    };
  
    // Jouw dataset met verschillende seizoenen en statistieken
    const dataBySeason = {
      2018: {
        "G": 16,
        "RshTD": 11,
      },
      2019: {
        "G": 13,
        "RshTD": 6,
      },
      2021: {
        "G": 13,
        "RshTD": 2,
      },
      2022: {
        "G": 16,
        "RshTD": 10,
      },
      // Voeg hier andere seizoenen en statistieken toe...
    };
  
    onMount(() => {
      createDonutChart();
    });
  
    afterUpdate(() => {
      createDonutChart();
    });
  
    function createDonutChart() {
      const width = 300;
      const height = 300;
      const margin = { top: 20, right: 20, bottom: 20, left: 20 };
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;
      const radius = Math.min(innerWidth, innerHeight) / 2;
  
      const svg = d3.select("#donut-chart").html("").append("svg")
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", `translate(${width / 2},${height / 2})`);
  
      const selectedData = dataBySeason[selectedSeason];
  
      const data = [selectedData[selectedStat], selectedData["RshTD"]];
  
      const color = d3.scaleOrdinal()
        .range(["#66c2a5", "#fc8d62"]); // Kleuren voor de geselecteerde statistiek en "Gs"
  
      const pie = d3.pie();
  
      const arc = d3.arc()
        .innerRadius(radius - 40)
        .outerRadius(radius);
  
      svg.selectAll("path")
        .data(pie(data))
        .enter()
        .append("path")
        .attr("d", arc)
        .attr("fill", (d, i) => color(i));
  
      // Voeg eventueel legenda toe
      const legend = svg.selectAll(".legend")
        .data([selectedStat, "RshTD"])
        .enter()
        .append("g")
        .attr("class", "legend")
        .attr("transform", (d, i) => `translate(0,${i * 20})`);
  
      legend.append("rect")
        .attr("x", 0)
        .attr("width", 18)
        .attr("height", 18)
        .style("fill", (d, i) => color(i));
  
      legend.append("text")
        .attr("x", 25)
        .attr("y", 9)
        .attr("dy", ".35em")
        .style("text-anchor", "start")
        .text(d => d);
    }
  
    // Functie om de data te filteren op basis van het seizoen en de geselecteerde statistiek
    function filterData(season, stat) {
      selectedSeason = season;
      selectedStat = stat;
  
      // Roep de functie aan om de donut chart bij te werken met de gefilterde data
      createDonutChart();
    }
  </script>
  
  <style>
    /* Voeg eventueel stijlen toe */
  </style>
  
  <div id="donut-chart"></div>
  
  <!-- Knoppen voor verschillende seizoenen -->
  <div>
    {#each Object.keys(dataBySeason) as season}
      <button on:click={() => filterData(season, selectedStat)}>{season}</button>
    {/each}
  </div>
  
  <!-- Vervolgkeuzemenu voor statistieken -->
  <select bind:value={selectedStat}>
    <option value="G">G</option>
    <option value="RshTD">RshTD</option>
    <!-- Voeg hier andere statistieken toe... -->
  </select>
  