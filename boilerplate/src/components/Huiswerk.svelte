<script>
  import { onMount, afterUpdate, tick } from "svelte";
  import * as d3 from "d3";
  import dataset from './data2012.json';

  const originalDataset = dataset;
  let currentPlayer = "Saquon Barkley";
  let selectedChart = "barchart";
  let donutChart1Data;
  let donutChart2Data;

  function updateDataset() {
    var filterdataset = originalDataset.filter((speler) => speler.Player === currentPlayer);
    filterdataset = filterdataset.filter((speler) => speler.AllTD !== "");
    return filterdataset;
  }

  function getColor(player) {
    const colorArray = {
      "Saquon Barkley": "#155ae6",
      "Arian Foster": "#e64747",
      "Adrian Peterson": "#008080",
      "Kareem Hunt": "#ff7b29",
      "Le'Veon Bell": "#e6a315"
    };

    return colorArray[player] || "#808080";
  }

  const margin = { top: 20, right: 20, bottom: 50, left: 180 };
  const width = 900 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;
  const maxTouchdowns = d3.max(originalDataset, (d) => d.AllTD);
  let svg;

  onMount(() => {
    svg = d3.select("#chart").append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    const knoppen = document.querySelectorAll(".knop");
    knoppen.forEach((knop) => {
      knop.addEventListener("click", (event) => {
        const selectedPlayer = event.target.textContent;
        changePlayer(selectedPlayer);
      });
    });
  });

  afterUpdate(() => {
    updateChart();
  });

  async function changePlayer(selectedPlayer) {
    currentPlayer = selectedPlayer;
    updateChart();
    await tick();
    updateImagePosition();
  }

  function updateImagePosition() {
    const img = document.querySelector("img");
    img.style.left = "50%";
    img.style.transform = "translateX(-50%)";
  }

  function updateChart() {
    const filterdataset = updateDataset();

    const xScale = d3.scaleLinear()
      .domain([0, d3.max(filterdataset, (d) => d.AllTD)])
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
      .attr("fill", (d) => getColor(d.Player))
      .merge(bars)
      .transition()
      .duration(1000)
      .attr("x", 0)
      .attr("width", (d) => xScale(d.AllTD))
      .attr("fill", (d) => getColor(d.Player));

    const labels = svg.selectAll("text")
      .data(filterdataset);

    labels.exit().remove();

    labels.enter().append("text")
      .text(0)
      .attr("x", -5)
      .attr("y", (d) => yScale(d.Season.toString()) + 20 / 1.5)
      .attr("text-anchor", "start")
      .attr("font-size", "15px")
      .attr("font-weight", "bold")
      .attr("fill", "white")
      .merge(labels)
      .transition()
      .duration(1000)
      .text((d) => d.AllTD)
      .attr("x", (d) => xScale(d.AllTD) - 30);

    console.log("Chart updated");

    if (selectedChart === "donutcharts") {
      createDonutCharts();
      animateDonutCharts();
    }
  }

  function animateDonutCharts() {
    d3.selectAll("#donut-chart1, #donut-chart2")
      .style("opacity", 0)
      .transition()
      .duration(1000)
      .style("opacity", 1);
  }
  
  function createDonutCharts() {
    const filterdataset = updateDataset();

    // Summing up AllTD across all seasons
    const totalAllTD = filterdataset.reduce((acc, season) => acc + Number(season.AllTD || 0), 0);

    // Assuming G, RshTD, and RecTD are values you want to include in the donut chart
    donutChart1Data = [
      { label: "AllTD", value: totalAllTD },
      { label: "G", value: filterdataset[0].G }
    ];

    donutChart2Data = [
      { label: "RshTD", value: filterdataset[0].RshTD },
      { label: "RecTD", value: filterdataset[0].RecTD }
    ];

    createDonutChart("#donut-chart1", donutChart1Data, "Totale Touchdowns vs Totale Games");
    createDonutChart("#donut-chart2", donutChart2Data, "Rushing Touchdowns vs Receiving Touchdowns");
  }

  
  function createDonutChart(containerId, dataForDonut, chartTitle) {
    const width = 400;
    const height = 400;
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

    const pieData = pie(dataForDonut.map(d => d.value));

    svgDonut.selectAll("path")
      .data(pieData)
      .enter()
      .append("path")
      .attr("d", arc)
      .attr("fill", (d, i) => color(i));

    // Centered value
    svgDonut.append("text")
      .attr("x", 0)
      .attr("y", 0)
      .attr("text-anchor", "middle")
      .attr("dy", "0.35em")
      .attr("font-size", "70px")
      .attr("font-weight", "bold")
      .attr("fill", "black")
      .text(dataForDonut[0].value);

    // Move legend outside the chart
    const legend = svgDonut.selectAll(".legend")
      .data(dataForDonut)
      .enter()
      .append("g")
      .attr("class", "legend")
      .attr("transform", (d, i) => `translate(${radius + 20},${(i - 1) * 20})`);

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
      .text(d => d.label);

    // svgDonut.append("text")
    // .attr("x", 0)
    // .attr("y", -radius - 10)
    // .attr("text-anchor", "middle")
    // .attr("fill", "black")

    console.log("Donut chart created");
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
    gap: 10px;
    margin-bottom: 200px;
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
    background-color: #155ae6;
    color: white;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #ff7b29;
  }

  img {
    max-width: 35%;
    position: absolute;
    top: 20%;
    margin-top: 60px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 2;
  }

  #chart, #donut-chart1, #donut-chart2 {
    width: 300px;
    height: 300px;
    margin: 10px;
  }
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