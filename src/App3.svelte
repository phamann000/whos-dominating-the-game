<script>
  import { onMount } from 'svelte';
  import { writable } from 'svelte/store';
  import { Line } from 'svelte-chartjs';
  import { Chart, registerables } from 'chart.js';
  import Papa from 'papaparse';
  import * as d3 from 'd3';
  import PublisherBubble from './PublisherBubble.svelte';



  Chart.register(...registerables);

  // Head-2-Head publishers combat data
  let publishers = [];
  let selectedPublisher1 = '';
  let selectedPublisher2 = '';
  let selectedMetric = 'total_sales';
  let chartData = writable({ datasets: [] });
  let data = [];

  // Genre data for horizontal bar charts
  let gamesData = [];
  let salesData = [];
  let selectedYear = 1980; 
  let svgGames, svgSales;
  let xScaleGames, yScaleGames, xScaleSales, yScaleSales;
  const margin = { top: 20, right: 30, bottom: 40, left: 100 };
  const width = 600 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;

  onMount(async () => {
    const response = await fetch('https://raw.githubusercontent.com/pndang/whos-dominating-the-game/main/data/phu-data.csv');
    const csvText = await response.text();
    Papa.parse(csvText, {
      header: true,
      dynamicTyping: true,
      complete: function(results) {
        data = results.data;
        publishers = Array.from(new Set(data.map(d => d.publisher)));
      },
    });

    // Parse games data
    Papa.parse('https://raw.githubusercontent.com/pndang/whos-dominating-the-game/main/data/games_per_genre_year.csv', {
      download: true,
      header: true,
      complete: function(results) {
        gamesData = results.data.map(d => ({
          year: +d.year,
          genre: d.genre,
          count: +d.Number_of_Games
        }));
        createGamesChart();
        updateGamesChart(gamesData, selectedYear);
      },
    });

    // Parse sales data
    Papa.parse('https://raw.githubusercontent.com/pndang/whos-dominating-the-game/main/data/sales_per_genre_year.csv', {
      download: true,
      header: true,
      complete: function(results) {
        salesData = results.data.map(d => ({
          year: +d.year,
          genre: d.genre,
          sales: +d.Total_Sales
        }));
        createSalesChart();
        updateSalesChart(salesData, selectedYear);
      },
    });
  });

  // Update Head-2-Head chart data
  $: if (selectedPublisher1 && selectedPublisher2) {
    const filteredData1 = data.filter(d => d.publisher === selectedPublisher1);
    const filteredData2 = data.filter(d => d.publisher === selectedPublisher2);

    const years = Array.from(new Set([...filteredData1.map(d => d.year), ...filteredData2.map(d => d.year)])).sort();

    const dataKey = selectedMetric;
    const label = selectedMetric === 'total_sales' ? 'mean total sales ($ millions)' : 'mean critic score (out of 10)';

    const salesData1 = years.map(year => {
      const entry = filteredData1.find(d => d.year === year);
      return entry ? { x: year, y: entry[dataKey], entry } : null;
    }).filter(d => d !== null);

    const salesData2 = years.map(year => {
      const entry = filteredData2.find(d => d.year === year);
      return entry ? { x: year, y: entry[dataKey], entry } : null;
    }).filter(d => d !== null);

    chartData.set({
      labels: years,
      datasets: [
        {
          label: selectedPublisher1,
          data: salesData1,
          borderColor: 'rgba(75, 192, 192, 1)',
          borderWidth: 2,
          fill: false,
          pointRadius: 5,
          pointHoverRadius: 10 // For better user experience
        },
        {
          label: selectedPublisher2,
          data: salesData2,
          borderColor: 'rgba(153, 102, 255, 1)',
          borderWidth: 2,
          fill: false,
          pointRadius: 5,
          pointHoverRadius: 10
        }
      ]
    });
  }

  function selectMetric(metric) {
    selectedMetric = metric;
  }

  function createGamesChart() {
    svgGames = d3
      .select('#games-chart')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left}, ${margin.top})`);

    xScaleGames = d3.scaleLinear().range([0, width]);
    yScaleGames = d3.scaleBand().range([height, 0]).padding(0.1);

    svgGames.append('text')
      .attr('class', 'y-axis-title')
      .attr('transform', `translate(${-margin.left + 22}, ${height / 2}) rotate(-90)`)
      .attr('text-anchor', 'middle')
      .attr('fill', '#ffcc00')
      .text('Genre');

    svgGames.append('text')
      .attr('class', 'x-axis-title')
      .attr('transform', `translate(${width / 2}, ${height + margin.bottom})`)
      .attr('text-anchor', 'middle')
      .attr('fill', '#ffcc00')
      .text('# of Games');
  }

  function updateGamesChart(data, year) {
    const filteredData = data.filter(d => d.year === year);

    filteredData.sort((a, b) => b.count - a.count);

    xScaleGames.domain([0, d3.max(filteredData, d => d.count)]);
    yScaleGames.domain(filteredData.map(d => d.genre));

    svgGames.selectAll('.bar').remove();
    svgGames.selectAll('.axis').remove();

    svgGames
      .append('g')
      .attr('transform', `translate(0, ${height})`)
      .attr('class', 'axis')
      .call(d3.axisBottom(xScaleGames).ticks(5).tickSize(0).tickPadding(6))
      .selectAll('text')
      .style('fill', '#e0e0e0');

    svgGames
      .append('g')
      .attr('class', 'axis')
      .call(d3.axisLeft(yScaleGames).tickSize(0).tickPadding(6))
      .selectAll('text')
      .style('fill', '#e0e0e0');

    svgGames.selectAll('.bar')
      .data(filteredData)
      .enter()
      .append('rect')
      .attr('class', 'bar')
      .attr('x', 0)
      .attr('y', d => yScaleGames(d.genre))
      .attr('width', 0)
      .attr('height', yScaleGames.bandwidth())
      .attr('fill', '#9966ff')
      .transition()
      .duration(1000) // Duration for smooth transition
      .attr('width', d => xScaleGames(d.count));
  }

  function createSalesChart() {
    svgSales = d3
      .select('#sales-chart')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left}, ${margin.top})`);

    xScaleSales = d3.scaleLinear().range([0, width]);
    yScaleSales = d3.scaleBand().range([height, 0]).padding(0.1);

    svgSales.append('text')
      .attr('class', 'y-axis-title')
      .attr('transform', `translate(${-margin.left + 22}, ${height / 2}) rotate(-90)`)
      .attr('text-anchor', 'middle')
      .attr('fill', '#ffcc00')
      .text('Genre');

    svgSales.append('text')
      .attr('class', 'x-axis-title')
      .attr('transform', `translate(${width / 2}, ${height + margin.bottom})`)
      .attr('text-anchor', 'middle')
      .attr('fill', '#ffcc00')
      .text('Sales ($ millions)');
  }

  function updateSalesChart(data, year) {
    const filteredData = data.filter(d => d.year === year);

    filteredData.sort((a, b) => b.sales - a.sales);

    xScaleSales.domain([0, d3.max(filteredData, d => d.sales)]);
    yScaleSales.domain(filteredData.map(d => d.genre));

    svgSales.selectAll('.bar').remove();
    svgSales.selectAll('.axis').remove();

    svgSales
      .append('g')
      .attr('transform', `translate(0, ${height})`)
      .attr('class', 'axis')
      .call(d3.axisBottom(xScaleSales).ticks(5).tickSize(0).tickPadding(6))
      .selectAll('text')
      .style('fill', '#e0e0e0');

    svgSales
      .append('g')
      .attr('class', 'axis')
      .call(d3.axisLeft(yScaleSales).tickSize(0).tickPadding(6))
      .selectAll('text')
      .style('fill', '#e0e0e0');

    svgSales.selectAll('.bar')
      .data(filteredData)
      .enter()
      .append('rect')
      .attr('class', 'bar')
      .attr('x', 0)
      .attr('y', d => yScaleSales(d.genre))
      .attr('width', 0)
      .attr('height', yScaleSales.bandwidth())
      .attr('fill', '#4bc0c0')
      .transition()
      .duration(1000) // Duration for smooth transition
      .attr('width', d => xScaleSales(d.sales));
  }

  function updateCharts() {
    updateGamesChart(gamesData, selectedYear);
    updateSalesChart(salesData, selectedYear);
  }

  function handleSliderInput(event) {
    selectedYear = +event.target.value;
    updateCharts();
  }
</script>

<style>
  :global(body) {
    background-color: #121212;
    color: #e0e0e0;
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
  }

  .slider-container {
    margin: 20px 0;
    display: flex;
    justify-content: center;
  }

  .slider {
    width: 300px;
  }

  .main-title {
    text-align: center;
    font-size: 28px;
    font-weight: bold;
    color: #e0e0e0;
    margin-bottom: 20px;
  }

  .main-title span {
    font-weight: bold;
  }

  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }

  .dropdowns {
    margin-bottom: 20px;
  }

  .dropdowns select {
    margin: 0 10px;
    background-color: #333;
    color: #e0e0e0;
    border: 1px solid #555;
    padding: 5px;
  }

  .chart-container {
    width: 70vw;
    height: 70vh;
  }

  h1 {
    margin-bottom: 10px;
    text-align: center;
    font-size: 24px;
    font-weight: bold;
    color: #e0e0e0;
  }

  .buttons {
    margin-bottom: 10px;
  }

  .buttons button {
    margin: 0 10px;
    padding: 10px 20px;
    background-color: #333;
    color: #e0e0e0;
    border: 1px solid #555;
    cursor: pointer;
  }

  .buttons button.active {
    background-color: #555;
  }

  .bar-chart-container {
    display: flex;
    justify-content: space-around;
    margin-bottom: -222px; 
  }

  .x-axis-title, .y-axis-title {
    font-size: 16px;
    fill: #ffcc00;
  }

  .axis path,
  .axis line {
    stroke: #e0e0e0;
  }

  p {
  text-align: left;
  }
</style>


<div class="main-title">
  <span>🎮 Who's Dominating the Game:</span> A Temporal Analysis of Gaming Publishers 🎮
</div>
<h4> by Annie Pham, Phu Dang, and Sean Chan</h4>
<br>

<h1>⚔️  Genre Prevalence Combat  ⚔️</h1>
<h3><em>What's your favorite genre?  🤔</em></h3>

<br>

<div class="slider-container">
  <input type="range" min="1980" max="2020" value={selectedYear} class="slider" on:input="{handleSliderInput}">
  <span>{selectedYear}</span>
</div>

<div class="bar-chart-container">
  <div id="games-chart" class="chart-container"></div>
  <div id="sales-chart" class="chart-container"></div>
</div>

<section>
  <br>
  <br>
  <br>
  <br>
  <h2 align="left"> <br>A publisher wants...</h2>
  <p>
    To create a brand! <br>
    The following bubble plot is of games published by Square Enix
  </p>
  <br>
  <PublisherBubble />
</section>

<div class="container">
  <h1>⚔️  Head-2-Head Publishers Combat  ⚔️</h1>
  <h3><em>How does your favorite publisher match up?  🔥🔥🔥</em></h3>
  
  <br>

  <div class="buttons">
    <button class:active={selectedMetric === 'total_sales'} on:click={() => selectMetric('total_sales')}>Mean Total Sales</button>
    <button class:active={selectedMetric === 'critic_score'} on:click={() => selectMetric('critic_score')}>Mean Crit Score</button>
  </div>

  <br>

  <div class="dropdowns">
    <select bind:value={selectedPublisher1}>
      <option value="">Select Publisher 1</option>
      {#each publishers as publisher}
        <option value={publisher}>{publisher}</option>
      {/each}
    </select>

    <select bind:value={selectedPublisher2}>
      <option value="">Select Publisher 2</option>
      {#each publishers as publisher}
        <option value={publisher}>{publisher}</option>
      {/each}
    </select>
  </div>

  <br>

  {#if $chartData.datasets.length > 0}
    <div class="chart-container">
      <Line 
        data={$chartData} 
        options={{
          responsive: true,
          maintainAspectRatio: true,
          scales: {
            x: {
              title: {
                display: true,
                text: 'Year',
                color: '#ffcc00',
                font: {
                  size: 22
                },
                padding: { top: 20 }
              },
              ticks: {
                color: '#e0e0e0' 
              }
            },
            y: {
              title: {
                display: true,
                text: selectedMetric === 'total_sales' ? 'mean total sales ($ millions)' : 'mean critic score (out of 10)',
                color: '#ffcc00', 
                font: {
                  size: 22
                },
                padding: { bottom: 20 }
              },
              ticks: {
                color: '#e0e0e0' 
              }
            }
          },
          plugins: {
            legend: {
              labels: {
                color: '#e0e0e0' 
              }
            },
            tooltip: {
              callbacks: {
                label: function(context) {
                  const entry = context.raw.entry;
                  return [
                    `   Year: ${context.label}`,
                    `   Mean ${selectedMetric.replace('_', ' ')}: ${context.raw.y}`,
                    `   Highest Sales: ${entry.highestSales}`,
                    `   Lowest Sales: ${entry.lowestSales}`,
                    `   Highest Critic Score: ${entry.highestScore}`,
                    `   Lowest Critic Score: ${entry.lowestScore}`
                  ];
                }
              },
              bodyColor: '#e0e0e0', 
              titleColor: '#e0e0e0', 
              titleFont: {
                weight: 'bold' 
              },
              bodyFont: {
                weight: 'normal'
              },
              displayColors: false,
              useHTML: true
            }
          },
          elements: {
            point: {
              radius: 5, 
              hoverRadius: 10, 
              hitRadius: 10 // Make hovering easier
            }
          }
        }} 
      />
    </div>
  {/if}
</div>

