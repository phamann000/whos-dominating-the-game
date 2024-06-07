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
    const totalGames = d3.sum(filteredData, d => d.count);

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

    const tooltip = d3.select('#tooltip');

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
      .on('mouseover', (event, d) => {
      tooltip.transition().duration(200).style('opacity', .9);
      tooltip.html(`${d.genre}: ${((d.count / totalGames) * 100).toFixed(2)}%`)
        .style('left', (event.pageX + 5) + 'px')
        .style('top', (event.pageY - 28) + 'px');
    })
        .on('mouseout', () => {
    tooltip.transition().duration(500).style('opacity', 0);
    })
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
    const totalSales = d3.sum(filteredData, d => +d.sales);

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

    const tooltip = d3.select('#tooltip');

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
      .on('mouseover', (event, d) => {
                        tooltip.transition().duration(200).style('opacity', .9);
                        tooltip.html(`${d.genre}: ${((+d.sales / totalSales) * 100).toFixed(2)}%`)
                            .style('left', (event.pageX + 5) + 'px')
                            .style('top', (event.pageY - 28) + 'px');
                    })
                    .on('mouseout', () => {
                        tooltip.transition().duration(500).style('opacity', 0);
                    })
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

<section>
  <p>
    Thousands of games are worked on and developed every month, so how do publishers decide which games they want to market and release? 
    Well, like any business, game publishers are out to make profit. Depending on the publisher behind a video game, they may decide to 
    help with or even shoulder development costs, take charge of marketing, and distribute the game, in return taking a portion of the profit
    the game ends up making. <br><br>

    But funding a game can be risky, depending on the size of the project and size of the team working on the project, 
    developing a game can take months to years. Take for example 2023's game of the year Baldur's Gate 3 a Role-Playing game which spent over 6 years in development
    and had over $100 million dollars in development cost, as much as many modern day Hollywood movies. <br><br>

    Although that game turned out wildly successful, most games can't even dream of reaching that level of success, so how would a publisher decide on whether
    or not they should risk funding a game? Well a good place to start is to look at trends from previous years! <br><br>

    <img src="src/assets/annotated_largesales_2012.png" alt="annotated_largesales_2012" width="550" height="400" class="center"><br>

    From 2011-2018, shooter games and sport games continously make it to the top 3 in sales. With a prime example of that being in 2012.
    Despite being 2 of more than 15 genres, Sport and Shooter games makeup over a third of sales combined across these 8 years.
    So, to optimize profits, publishers should just release Sport and Shooter games, right? Well... it's a bit more complicated than that, but feel
    to try to find your own trends! What type of game would you want to fund if you were a publisher?
  </p>
</section>
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
  <div id="tooltip" style="position: absolute; opacity: 0; background-color: #333; color: #fff; padding: 5px; border-radius: 3px; pointer-events: none;"></div>
</div>

<section>
  <br>
  <br>
  <br>
  <br>
  <h2 align="left"> <br>A publisher wants...</h2>
  <p>
    To create a brand! Looking at numbers from a holistic view may actually lead to some devastating misunderstandings. Take the bar chart
    we went out of our way to show you earlier, it'd be easy to say that creating Shooter games leads to great profits, but in actuality,
    a compartively small number of shooter games were actually released in 2012! 
    
    <br> <br>(use our interaction above to see just how few!)<br><br>
    
    In fact, we only have to look at one publisher to see just where these huge sales are coming from. Activision. <br> <br>

    Activision has released a shooter game almost every year we see shooter games beat out other genres in sales. Even in 2012 when they released
    Call of Duty: Black Ops 2 which has almost 30 millions dollars in sale, just about a third of the total sales shooter games made in that year!
    So it's not that shooter games lead to huge sales like the initial data may have led you to believe, but rather Activison, a company famous
    for their shooter games, continously put out these games to make huge profits. <br><br>

    That's what it means to create a brand. Many video game players stick with playing the same handful of games with the same small subset of genres,
    just like how readers may keep reading Stephen King for horror books or Terry Pratchett for fantasy games, you'll go to Activison for shooter games. <br><br>

    We can even see this trend for other game genres. Choosing a publisher from the top 100 in sales will show you the games that publisher has released, color-coded
    for genre and sized by sales. Take Square Enix, for example, and we can see that this publisher overwhemlingly puts out Role-Playing games as part of their infamous 
    Final Fantasy Series which as of writing this has 16 installments in the main series alone. <br><br>

    Creating a brand doesn't just help consumers, but the publishers themselves. Knowing that the developers they're hiring can put out good games of this genre
    means like it will like less of an upfront risk, afterall, they've already succeeded in making a game like this, chances are they can make another.
    There's also less of an upfront cost as, hypothetically, many skills and even assets can be reused or transferred over from one game to the next meaning developers 
    won't have to start from scratch each time they want to release another game. 


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

