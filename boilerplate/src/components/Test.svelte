<script>
  import { onMount, afterUpdate, tick } from "svelte";
  import * as d3 from "d3";
  import dataset from './data2012.json';

  const originalDataset = dataset;
  let currentPlayer = "Saquon Barkley";
  let selectedChart = "barchart";
  let donutChart1Data;
  let donutChart2Data;

  //Hier maak ik een functie aan die ik later kan hergebruiken. er word gefilterd op 'speler' uit data2012.json en de AllTD//
  function updateDataset() {
    var filterdataset = originalDataset.filter((speler) => speler.Player === currentPlayer);
    filterdataset = filterdataset.filter((speler) => speler.AllTD !== "");
    return filterdataset;
  }
  

//hier maak ik een functie om de kleuren aan te passen van de barchart per speler later//
  function getColor(player) {
    const colorArray = {
      "Saquon Barkley": "#155ae6",
      "Arian Foster": "#e64747",
      "Adrian Peterson": "#008080",
      "Kareem Hunt": "#ff7b29",
      "Le'Veon Bell": "#e6a315",
      "AllTD": "#155ae6",
      "G": "#ff0000"
    };

    return colorArray[player] || "#808080";
  }

  function createColorScale(selectedPlayer) {
    const colorArray = {
      "Saquon Barkley": "#155ae6",
      "Arian Foster": "#e64747",
      "Adrian Peterson": "#008080",
      "Kareem Hunt": "#ff7b29",
      "Le'Veon Bell": "#e6a315"
    };

    const colorScale = d3.scaleOrdinal()
      .domain(Object.keys(colorArray))
      .range(Object.values(colorArray));

    return colorScale(selectedPlayer);
  }

//margins voor de svg//
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

//Na de update word updatechart opgeroepen opnieuw //
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

//bars worden bij de functie updatechart telkens opnieuw getekend en verwijderd//
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

//bars worden hier verwijderd//      
    bars.exit().remove();

//bars worden hier getekend//    
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

//labels worden verwijderd//      
    labels.exit().remove();

//labels worden getekend hier//    
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

//als donut chart geselecteerd worden word pas de donutchart gecreeert//    
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

  // hier telt hij alle totale touchdowns op van het seizoen bij elkaar//
  const totalAllTD = filterdataset.reduce((acc, season) => acc + Number(season.AllTD || 0), 0);

  // donutchart1 krijgt hier de kleuren die worden toegewezen bij de if statement//
  donutChart1Data = [
    { label: "AllTD", value: totalAllTD, color: getColor("AllTD") },
    { label: "G", value: filterdataset[0].G, color: getColor("G") }
  ];

  // kleur toepassen per speler//
  if (currentPlayer === "Saquon Barkley") {
    donutChart1Data[0].color = "#155ae6"; // Blue for "AllTD"
    donutChart1Data[1].color = "#ff0000"; // Red for "G"
  } else if (currentPlayer === "Arian Foster") {
    donutChart1Data[0].color = "#e64747"; // Red for "AllTD"
    donutChart1Data[1].color = "#00008b"; // Dark blue for "G"
  } else if (currentPlayer === "Adrian Peterson") {
    donutChart1Data[0].color = "#008080"; // Teal for "AllTD"
    donutChart1Data[1].color = "#008000"; // Green for "G"
  } else if (currentPlayer === "Kareem Hunt") {
    donutChart1Data[0].color = "#ff7b29"; // Orange for "AllTD"
    donutChart1Data[1].color = "#008000"; // Green for "G"
  } else if (currentPlayer === "Le'Veon Bell") {
    donutChart1Data[0].color = "#e6a315"; // Gold for "AllTD"
    donutChart1Data[1].color = "#008000"; // Green for "G"
  };

  //zelfde voor donutchart 2//
  donutChart2Data = [
    { label: "RshTD", value: filterdataset[0].RshTD, color: getColor("RshTD") },
    { label: "RecTD", value: filterdataset[0].RecTD, color: getColor("RecTD") }
  ];

  if (currentPlayer === "Saquon Barkley") {
    donutChart2Data[0].color = "#ff0000"; // Red for "RshTD"
    donutChart2Data[1].color = "#155ae6"; // Blue for "RecTD"
  } else if (currentPlayer === "Arian Foster") {
    donutChart2Data[0].color = "#00008b"; // Dark blue for "RshTD"
    donutChart2Data[1].color = "#e64747"; // Red for "RecTD"
  } else if (currentPlayer === "Adrian Peterson") {
    donutChart2Data[0].color = "#008000"; // Green for "RshTD"
    donutChart2Data[1].color = "#008080"; // Teal for "RecTD"
  } else if (currentPlayer === "Kareem Hunt") {
    donutChart2Data[0].color = "#008000"; // Green for "RshTD"
    donutChart2Data[1].color = "#ff7b29"; // Orange for "RecTD"
  } else if (currentPlayer === "Le'Veon Bell") {
    donutChart2Data[0].color = "#008000"; // Green for "RshTD"
    donutChart2Data[1].color = "#e6a315"; // Gold for "RecTD"
  };

  //createdonutchart word aangeroepen//
  createDonutChart("#donut-chart1", donutChart1Data, "AllTD vs G");
  createDonutChart("#donut-chart2", donutChart2Data, "RshTD vs RecTD");
}

//functie om de donut chart te tekenen//
  function createDonutChart(containerId, dataForDonut, chartTitle) {
    const width = 500;
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

    const selectedPlayerColor = createColorScale(currentPlayer);

    const color = d3.scaleOrdinal()
      .range([selectedPlayerColor, "#fc8d62"]); // Update the color range

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
      .attr("fill", (d, i) => dataForDonut[i].color); // Use dynamic color from data

    // de waarde die binnen word weergeven, alle touchdowns bij elkaar opgeteld.
    svgDonut.append("text")
      .attr("x", 0)
      .attr("y", 0)
      .attr("text-anchor", "middle")
      .attr("dy", "0.35em")
      .attr("font-size", "70px")
      .attr("font-weight", "bold")
      .attr("fill", "black")
      .text(dataForDonut[0].value);

    // legenda word hier getekend 
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
      .style("fill", (d, i) => dataForDonut[i].color); 

    legend.append("text")
      .attr("x", 25)
      .attr("y", 9)
      .attr("dy", ".35em")
      .style("text-anchor", "start")
      .text(d => d.label);

    svgDonut.append("text")
      .attr("x", 0)
      .attr("y", -radius - 10)
      .attr("text-anchor", "middle")
      .text(chartTitle);

    console.log("Donut chart created");
  }
</script>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 90vh;
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
    background-color: #4c7aaf;
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
    left: 50%;
    transform: translateX(-50%);
    z-index: 2;
  }

  #chart, #donut-chart2 {
    width: 300px;
    height: 300px;
    margin: 10px;
  }

  #donut-chart1{
    margin-left: -200px;
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
