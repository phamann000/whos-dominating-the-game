
<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { pb_data } from "./assets/pb_data.js";

  let data = pb_data;
  let nodes;
  const forceStrength = 0.03;

  onMount(() => {

    const bub_rad = d3.scaleLinear([0.01, 64.29], [10, 100]);
    const color = d3.scaleOrdinal([
      'Action', 'Action-Adventure', 'Adventure', 'Board Game','Education', 
      'Fighting', 'MMO', 'Misc', 'Music', 'Party',
       'Platform', 'Puzzle', 'Racing', 'Role-Playing', 'Shooter',
       'Simulation', 'Sports', 'Strategy', 'Visual Novel'], 
       ["#C40700", "#E25A00", "#FFF34F", "#6DE055", "#62B5CE",
        "#6D0505", "#51B160", "#660F7A", "#005AA8", "#FFD1DC",
        "#DAD47C", "#A67FEC", "#FFA433", "#CC3B94", "#CD7F32",
        "#D152ED", "#E3DB00", "#02175B", "#DF7D97"
       ]);

  
    const categories = ['Square Enix', '2K Games', '2K Play', '2K Sports', '3DO', '505 Games',
       '989 Studios', 'ASC Games', 'ASCII Entertainment',
       'Acclaim Entertainment', 'Accolade', 'Activision', 'Agetec',
       'Aksys Games', 'Aspyr', 'Atari', 'Atlus', 'BAM! Entertainment',
       'Bandai', 'Banpresto', 'Bethesda Softworks',
       'Blizzard Entertainment', 'Buena Vista', 'Capcom',
       'City Interactive', 'Codemasters', 'Conspiracy Entertainment',
       'Crave Entertainment', 'D3 Publisher', 'DSI Games', 'Deep Silver',
       'Destination Software, Inc', 'Destineer',
       'Disney Interactive Studios', 'EA Sports', 'EA Sports BIG',
       'Eidos Interactive', 'Electronic Arts', 'Empire Interactive',
       'Enix', 'Focus Home Interactive', 'GT Interactive',
       'Gathering of Developers', 'Global Star Software', 'Gotham Games',
       'Hasbro Interactive', 'Hudson Soft', 'Idea Factory', 'Imagic',
       'Infogrames', 'Interplay', 'Jaleco', 'KOEI', 'Konami',
       'Konami Digital Entertainment', 'Level 5', 'LucasArts',
       'MTV Games', 'Majesco', 'Mastiff', 'Mattel Interactive',
       'Microsoft', 'Microsoft Game Studios', 'Microsoft Studios',
       'Midway Games', 'NIS America', 'Namco', 'Namco Bandai',
       'Namco Bandai Games', 'Natsume', 'Nintendo', 'RedOctane',
       'Rockstar Games', 'Sega', 'Sierra Entertainment',
       'Sony Computer Entertainment',
       'Sony Computer Entertainment America',
       'Sony Interactive Entertainment', 'Sony Online Entertainment',
       'SouthPeak Interactive', 'Square', 'Square EA',
       'TDK Mediactive', 'THQ', 'Take-Two Interactive', 'Tecmo',
       'Tecmo Koei', 'Telltale Games', 'Tengen', 'Tomy Corporation',
       'Ubisoft', 'Universal Interactive', 'Unknown', 'VU Games',
       'Virgin Interactive', 'Vivendi Games', 'Warner Bros. Interactive',
       'Warner Bros. Interactive Entertainment', 'Xseed Games',
       'Zoo Games'];
    // Inital Variables
    const svg = d3.select('svg');
    const width = +svg.attr('width');
    const height = +svg.attr('height');
    let selectedCategory = "Square Enix";
    const center = {x: width / 2, y: height/ 2};
    //CREATE LEGEND
    const legend = svg.selectAll(".legend")
     .data([
      'Action', 'Action-Adventure', 'Adventure', 'Board Game','Education', 
      'Fighting', 'MMO', 'Misc', 'Music', 'Party',
       'Platform', 'Puzzle', 'Racing', 'Role-Playing', 'Shooter',
       'Simulation', 'Sports', 'Strategy', 'Visual Novel'])
      .enter().append("g")
     .attr("class", "legend")
     .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; });

    // draw legend colored rectangles
    legend.append("rect")
     .attr("x", 0)
     .attr("width", 18)
     .attr("height", 18)
     .style("fill", function(d){return color(d)});

    // draw legend text
    legend.append("text")
     .attr("x", 50)
     .attr("y", 9)
     .attr("dy", ".35em")
     .attr("stroke", 'white')
     .attr("fill", 'white')
     //.style("text-anchor", "end")
     .text(function(d) { return d;});

    update(selectedCategory);

    function update(selectedCategory) {
      // Filter data based on dropdown selection
      let filteredData = data.filter(d => d.publisher === selectedCategory);
      let nodes = filteredData.map(d => Object.create(d));

      // Remove SVG circles
      svg.selectAll('circle').remove();

      // Create simulation with filtered data
    
     let simulation = d3.forceSimulation(nodes)
        .force("center", d3.forceCenter(center.x, center.y))
        .force('x', d3.forceX().strength(forceStrength).x(center.x))
        .force('y', d3.forceY().strength(forceStrength).y(center.y))
        .force("collide", d3.forceCollide().radius(d => bub_rad(d.total_sales)))
        .force("charge", d3.forceManyBody().strength(-3))
        .on("tick", ticked);

      function ticked() {
      circles.attr("cx", d => d.x)
      .attr("cy", d => d.y);
      }

      const tooltip = d3.select('#tooltip');

      // Update new circles
      let circles = svg.selectAll('circle')
      .data(nodes)
      .enter()
      .append('circle')
      .attr('class', 'circle')
      .attr('r', d => bub_rad(d.total_sales))
      //.attr('r', d => Math.sqrt(d.sales) * 100)
      .attr('stroke', "white")
      .attr('fill', d => color(d.genre))
      .on('mouseover', (event, d) => {
                        tooltip.transition().duration(200).style('opacity', .9);
                        tooltip.html(`Game Title: ${d.title} <br>
                                      Sales (millions): $${d.total_sales} <br>
                                      Genre: ${d.genre} <br>
                                      Publisher: ${d.publisher} <br>
                                      Year: ${d.year}`)
                            .style('left', (event.pageX + 5) + 'px')
                            .style('top', (event.pageY - 28) + 'px');
                    })
                    .on('mouseout', () => {
                        tooltip.transition().duration(500).style('opacity', 0);
                    }); // Duration for smooth transition
    }

    //When drop down changes, grab new value in dropdown and update
    function handleCategoryChange(event) {
      selectedCategory = event.target.value;
      update(selectedCategory);
    }

    //Create dropdownwith all publisher options
    const dropdown = d3.select('#dropdown');

    dropdown.selectAll('option')
      .data(categories)
      .enter().append('option')
      .text(d => d)
      .attr('value', d => d);

    // Detect Change
    dropdown.on('change', handleCategoryChange);
  });
  
</script>

<svg width="1300" height="600"></svg>

<select id="dropdown">
  <!-- Dropdown options will be inserted here -->
  <div id="tooltip" data-html="true" style="text-align: left; position: left; opacity: 0; background-color: #333; color: #fff; padding: 5px; border-radius: 3px; pointer-events: none;"></div>
</select>

<style>
</style>