
<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import {
  forceCollide
} from "d3-force";
import { square_enix_data } from "./assets/square_enix_data.js"

  

  onMount(() => {
    const bub_rad = d3.scaleLinear([0, 64.29], [10, 80]);
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
    // D3 code for creating the bubble plot
    const svg = d3.select('svg');
    const width = +svg.attr('width');
    const height = +svg.attr('height');

    let data = square_enix_data
    let nodes = data.map(d => Object.create(d))

    const center = {x: width / 2, y: height/ 2};
    const forceStrength = 0.03;

    const simulation = d3.forceSimulation(nodes)
      .force("charge", d3.forceManyBody())
      .force("center", d3.forceCenter(center.x, center.y))
      .force("collide", forceCollide().radius(d => bub_rad(d.total_sales)))
      //.force("collide", forceCollide().radius(d => Math.sqrt(d.sales) * 100))
      .force('x', d3.forceX().strength(forceStrength).x(center.x))
      .force('y', d3.forceY().strength(forceStrength).y(center.y));

    const circles = svg.selectAll('circle')
      .data(nodes)
      .enter()
      .append('circle')
      .attr('r', d => bub_rad(d.total_sales))
      //.attr('r', d => Math.sqrt(d.sales) * 100)
      .attr('fill', d => color(d.genre));
      //.attr('fill', 'steelblue');

    simulation.on("tick", () => {
      circles.attr("cx", d => d.x)
      .attr("cy", d => d.y);
    });
  });
</script>

<svg width="500" height="500"></svg>
