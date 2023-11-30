<script>
    import { onMount, afterUpdate } from "svelte";
    import * as d3 from "d3";
    import BarChart from "./BarChart.svelte";
    import DonutChart from "./DonutChart.svelte";
    import dataset from './data2012.json';
  
    // Originele dataset
    const originalDataset = dataset;
  
    // Svelte state variabele voor de huidige speler
    let currentPlayer = "Saquon Barkley";
  
    // Geselecteerde grafiek (barchart, donutcharts)
    let selectedChart = "barchart";
  
    // Functie om de dataset voor de huidige speler te filteren
    function updateDataset() {
      var filterdataset = originalDataset.filter((speler) => speler.Player === currentPlayer);
      filterdataset = filterdataset.filter((speler) => speler.AllTD !== "");
      return filterdataset;
    }
  
    // Aangepaste grootte van het diagram
    const margin = { top: 20, right: 20, bottom: 50, left: 50 };
    const width = 500 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;
  
    let svg;
  
    onMount(() => {
      // Initialiseren van de SVG-container
      svg = d3.select("#chart").append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
  
      const knoppen = document.querySelectorAll(".knop");
      knoppen.forEach((knop) => {
        // Gebruik Svelte's on:click directive om de functie aan te roepen
        knop.addEventListener("click", (event) => {
          const selectedPlayer = event.target.textContent;
          changePlayer(selectedPlayer);
        });
      });
    });
  
    afterUpdate(() => {
      // Roep de update functie op bij het bijwerken van de component (bijvoorbeeld bij het veranderen van de huidige speler)
      updateChart();
    });
  
    function changePlayer(selectedPlayer) {
      // Update the currentPlayer variable
      currentPlayer = selectedPlayer;
  
      // Call the functions to update the chart
      updateChart();
    }
  
    function updateChart() {
      // Haal de gefilterde dataset op voor de huidige speler
      const filterdataset = updateDataset();
  
      // Haal de maximale waarde van AllTD in de gefilterde dataset op
      const maxTouchdowns = d3.max(filterdataset, (d) => +d.AllTD);
  
      // Maak een schaal voor de x-as (nu een lineaire schaal voor de horizontale weergave)
      const xScale = d3.scaleLinear()
        .domain([0, maxTouchdowns])
        .range([0, width]);
  
      // Maak een schaal voor de y-as
      const yScale = d3.scaleBand()
        .domain(filterdataset.map((d) => d.Season.toString()))
        .range([0, height])
        .padding(0.4); // Probeer een hogere waarde voor padding
  
      // Selecteer alle bestaande bars en definieer een overgang voor de update
      const bars = svg.selectAll("rect")
        .data(filterdataset);
  
      // Verwijder oude bars die niet meer nodig zijn
      bars.exit().remove();
  
      // Voeg nieuwe bars toe
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
  
      // Rest van de code blijft hetzelfde...
  
      // Update de donutcharts indien geselecteerd
      if (selectedChart === "donutcharts") {
        createDonutCharts();
      }
    }
  
    /////////////////////DONUT CHART//////////////////////////////
    function createDonutCharts() {
      // Haal de gefilterde dataset op voor de huidige speler
      const filterdataset = updateDataset();
  
      // Data voor de eerste donutchart
      const donutChart1Data = [filterdataset[0].AllTD, filterdataset[0].G];
  
      // Data voor de tweede donutchart
      const donutChart2Data = [filterdataset[0].RshTD, filterdataset[0].RecTD];
  
      // Creëer de eerste donutchart
      createDonutChart("#donut-chart1", donutChart1Data, "AllTD vs G");
  
      // Creëer de tweede donutchart
      createDonutChart("#donut-chart2", donutChart2Data, "RshTD vs RecTD");
    }
  
    function createDonutChart(containerId, dataForDonut, chartTitle) {
      const width = 300;
      const height = 300;
      const margin = { top: 20, right: 20, bottom: 20, left: 20 };
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;
      const radius = Math.min(innerWidth, innerHeight) / 2;
  
      const svgDonut = d3.select(containerId).html("").append("svg")
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", `translate(${width / 2},${height / 2})`);
  
      const color = d3.scaleOrdinal()
        .range(["#66c2a5", "#fc8d62"]);
  
      const pie = d3.pie();
  
      const arc = d3.arc()
        .innerRadius(radius - 40)
        .outerRadius(radius);
  
      svgDonut.selectAll("path")
        .data(pie(dataForDonut))
        .enter().append("path")
        .attr("d", arc)
        .attr("fill", d => color(d.data))
        .append("title")
        .text(d => `${d.data}`);
  
      svgDonut.append("text")
        .attr("text-anchor", "middle")
        .attr("font-size", "16px")
        .attr("dy", "0.35em")
        .text(chartTitle);
    }
  </script>
  
  <style>
    main {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
    }
  
    .content {
      display: flex;
      flex-direction: row;
      justify-content: space-around;
      width: 100%;
    }
  
    .knoppen {
      display: flex;
      flex-direction: row;
      gap: 10px; /* Voegt wat ruimte tussen de knoppen toe */
      margin-bottom: 200px; /* Geeft wat ruimte onder de knoppen */
    }
  
    select {
      margin-bottom: 20px;
    }
  
    button {
      padding: 10px 15px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      background-color: #4c7aaf; /* Groene kleur, je kunt deze aanpassen */
      color: white;
      transition: background-color 0.3s ease;
    }
  
    button:hover {
      background-color: #45a049;
    }
  
    img {
      max-width: 35%;
      position: absolute;
      top: 20%;
      margin-top: 60px;
      left: 35%;
      transform: translateX(-50%);
      z-index: 2; /* Zet de z-index hoger dan die van .content */
    }
  
    #chart, #donut-chart1, #donut-chart2 {
      width: 300px;
      height: 300px;
      margin: 10px;
    }
  
    /* Voeg andere stijlen toe zoals nodig */
  </style>
  
  <main>
    <h1>{currentPlayer.toUpperCase()}</h1>
    <div class="knoppen">
      <select bind:value={selectedChart}>
        <option value="barchart">Bar Chart</option>
        <option value="donutcharts">Donut Charts</option>
      </select>
      <button class="knop" on:click={() => changePlayer("Saquon Barkley")}>Saquon Barkley</button>
      <button class="knop" on:click={() => changePlayer("Arian Foster")}>Arian Foster</button>
      <button class="knop" on:click={() => changePlayer("Adrian Peterson")}>Adrian Peterson</button>
      <button class="knop" on:click={() => changePlayer("Kareem Hunt")}>Kareem Hunt</button>
      <button class="knop" on:click={() => changePlayer("Le'Veon Bell")}>Le'Veon Bell</button>
    </div>
    <img src="images/{currentPlayer}.png" alt=""/>
    <div class="content">
      {#if selectedChart === "barchart"}
        <div id="chart"></div>
      {:else if selectedChart === "donutcharts"}
        <div id="donut-chart1"></div>
        <div id="donut-chart2"></div>
      {/if}
    </div>
  </main>
  