
<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { square_enix_data } from "./assets/square_enix_data.js";

  

  onMount(() => {
    const bub_rad = d3.scaleLog([0.01, 64.29], [5, 30]);
    const color = d3.scaleOrdinal([
      'Action', 'Action-Adventure', 'Adventure', 'Board Game','Education', 
      'Fighting', 'MMO', 'Misc', 'Music', 'Party',
       'Platform', 'Puzzle', 'Racing', 'Role-Playing', 'Shooter',
       'Simulation', 'Sports', 'Strategy', 'Visual Novel'], 
       ["#55E6BA", "#55B3E6", "#E46F29", "#CCCA5F", "#506484",
        "#6B78B9", "#BE5ABD", "#B2E7DE", "#CEB33F", "#50E19F",
        "#C89F04", "#EE3F8D", "#7BAC13", "#3F6D05", "#608AE5",
        "#55E1E6", "#55E688", "#5585E6", "#A1E3E6"
       ]);
    const categories = ['Square Enix', 'Magical Company'];
    // D3 code for creating the bubble plot
    const svg = d3.select('svg');
    const width = +svg.attr('width');
    const height = +svg.attr('height');
    let selectedCategory;

    
    let data = square_enix_data;
    let nodes = data.map(d => Object.create(d));

    const center = {x: width / 2, y: height/ 2};
    const forceStrength = 0.03;

    let simulation = d3.forceSimulation(nodes)
      .alpha(0.5)
      .alphaTarget(0.3)
      .alphaDecay(0.02)
      .force("center", d3.forceCenter(center.x, center.y))
      .force('x', d3.forceX().strength(forceStrength).x(center.x))
      .force('y', d3.forceY().strength(forceStrength).y(center.y))
      .force("collide", d3.forceCollide().radius(d => bub_rad(d.total_sales)))
      //.force("collide", d3.forceCollide().radius(d => Math.sqrt(d.sales)))
      .force("charge", d3.forceManyBody().strength(-8))
      .on("tick", ticked);

    const circles = svg.selectAll('circle')
      .data(nodes)
      .enter()
      .append('circle')
      .attr('r', d => bub_rad(d.total_sales))
      //.attr('r', d => Math.sqrt(d.sales) * 100)
      .attr('stroke', "grey")
      .attr('fill', d => color(d.genre));
      //.attr('fill', 'steelblue');

    function ticked() {
      //console.log(nodes[10]['x'])
      circles.attr("cx", d => d.x)
      .attr("cy", d => d.y);
    }

    function update(selectedCategory) {
      // Filter data based on dropdown selection
      let filteredData = data.filter(d => d.publisher === selectedCategory);
      let filteredNodes = filteredData.map(d => Object.create(d));

      // Remove SVG circles
      svg.selectAll('circle').remove();
      
      simulation.alpha(1).restart(); 

      // Update simulation with filtered data
      simulation
        .force("center", d3.forceCenter(center.x, center.y))
        .force('x', d3.forceX().strength(forceStrength).x(center.x))
        .force('y', d3.forceY().strength(forceStrength).y(center.y))
        .force("collide", d3.forceCollide().radius(d => bub_rad(d.total_sales)))
        .force("charge", d3.forceManyBody().strength(-8))
        .on("tick", ticked);


      
      // Update new circles
      svg.selectAll('circle')
        .data(filteredNodes)
        .enter()
        .append('circle')
        .attr('r', d => bub_rad(d.total_sales))
        .attr('stroke', "grey")
        .attr('fill', d => color(d.genre));

    }

    function handleCategoryChange(event) {
      selectedCategory = event.target.value;
      update(selectedCategory);
    }

    const dropdown = d3.select('#dropdown');

    dropdown.selectAll('option')
      .data(categories)
      .enter().append('option')
      .text(d => d)
      .attr('value', d => d);

    dropdown.on('change', handleCategoryChange);
  });
</script>

<svg width="500" height="500"></svg>

<select id="dropdown">
  <!-- Dropdown options will be inserted here -->
</select>
