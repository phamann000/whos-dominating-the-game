
<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { pb_data } from "./assets/pb_data.js";

  let data = pb_data;
  let nodes;
  const forceStrength = 0.03;

  onMount(() => {

    const bub_rad = d3.scaleLog([0.01, 64.29], [5, 30]);
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

  
    const categories = ['Square Enix','10TACLE Studios', '11 bit studios', '1C Company', '2K Games',
       '2K Play', '2K Sports', "3 O'Clock", '3DO', '49Games', '505 Games',
       '5pb', '704Games', '7G//AMES', '8-4', 'A1 Games', 'AIA',
       'AQ Interactive', 'ARUSH Entertainment', 'ASCII Entertainment',
       'ASCII Media Works', 'Abel', 'Acclaim Entertainment', 'Accolade',
       'Acquire', 'Activision', 'Activision Value', 'Acttil', 'Aerosoft',
       'Agatsuma Entertainment', 'Agetec', 'Aksys Games',
       'Alawar Entertainment', 'Alchemist', 'Alliance Digital Media',
       'Alternative Software', 'Alvion', 'Anuman', 'Aqua Plus',
       'Aquaplus', 'Arc System Works', 'ArenaNet', 'Aria', 'Arika',
       'ArtDink', 'Ascaron Entertainment', 'Asgard',
       'Asmik Ace Entertainment', 'Aspyr', 'Assemble Entertainment',
       'Astragon', 'Asylum Entertainment', 'Atari', 'Athena', 'Atlus',
       'Avanquest', 'Avanquest Software', 'BAM! Entertainment',
       'Badland Games', 'Badland Studio', 'Bandai',
       'Bandai Namco Entertainment', 'Bandai Namco Games', 'Banpresto',
       'Benesse', 'Bergsala Lightweight', 'Berkeley', 'Best Media',
       'Bethesda Softworks', 'Big Ben Interactive', 'Big Fish Games',
       'Bigben Interactive', 'Black Bean Games', 'Black Label Games',
       'Blast! Entertainment Ltd', 'Blizzard Entertainment', 'Bold Games',
       'Boost On', 'Brash Entertainment', 'Broccoli', 'Buena Vista',
       'BushiRoad', 'CDV Software Entertainment', 'CK Games', 'Capcom',
       'Carbine Studios', 'Cave', 'ChunSoft', 'City Interactive',
       'Codemasters', 'CokeM Interactive', 'Comfort', 'Commseed',
       'Compile', 'Compile Heart', 'Conspiracy Entertainment',
       'Crave Entertainment', 'Creative Core', 'Crimson Cow',
       'Culture Brain', 'CyberFront', 'Cygames', 'D3 Publisher',
       'D3Publisher', 'DSI Games', 'DTP Entertainment', 'Daedalic',
       'Daedalic Entertainment', 'Daito', 'Datam Polystar', 'Deep Silver',
       'Destination Software, Inc', 'Destineer', 'Detn8 Games',
       'Devolver Digital', 'DigiCube', 'Digiturbo',
       'Dimple Entertainment', 'Disney Interactive Studios',
       'Dispatch Games', 'Dorart', 'DotEmu', 'Dovetail Games',
       'Dramatic Create', 'DreamCatcher Interactive',
       'Dusenberry Martin Racing', 'EA Sports', 'EA Sports BIG', 'ESP',
       'Easy Interactive', 'Ecole', 'Edia', 'Egosoft',
       'Eidos Interactive', 'El Dia', 'Electronic Arts',
       'Empire Interactive', 'Encore', 'Enix', 'Enlight', 'Enterbrain',
       'Entergram', 'Epic Games', 'Essential Games', 'EuroVideo Medien',
       'Evolution Games', 'Evolved Games', 'Excalibur Publishing',
       'Excalibur Publishing Limited', 'Experience Inc.',
       'FDG Entertainment', 'Falcom Corporation', 'Fields',
       'First Class Simulations', 'Flight-Plan', 'Focus Home Interactive',
       'Focus Multimedia', 'Foreign Media Games', 'From Software',
       'Frontier Developments', 'Frozenbyte', 'FuRyu Corporation',
       'Funbox Media', 'Funcom', 'G.Rev', 'GMX Media', 'GN Software',
       'GSP', 'Gadget Soft', 'Gaijinworks', 'Gakken', 'Game Factory',
       'GameMill', 'GameMill Entertainment', 'GameTrust', 'Gamecock',
       'Gameloft', 'Games Workshop', 'Gathering of Developers',
       'Gearbox Software', 'Genius Products, Inc.', 'Genki',
       'Genterprise', 'Ghostlight', 'Giants Software', 'Giga', 'Giza10',
       'Global A Entertainment', 'Global Star Software',
       'Got Game Entertainment', 'Gotham Games', 'Graffiti',
       'Grand Prix Games', 'Graphsim Entertainment', 'Grey Box',
       'Griffin International', 'Groove Games', 'Gun Media', 'GungHo',
       'GungHo Online Entertainment', 'Gust', 'Hackberry',
       'Hamster Corporation', 'Happinet', 'Harmonix Music Systems',
       'Harukaze', 'Hasbro Interactive', 'Headup Games',
       'Her Interactive', 'Hip Interactive',
       'Home Entertainment Suppliers', 'Hoplite Research', 'Hot-B',
       'Hudson Entertainment', 'Hudson Soft', 'HuneX', 'IE Institute',
       'IGS', 'Idea Factory', 'Idea Factory International',
       'Ignition Entertainment', 'Image Epoch', 'Imagineer',
       'Independent Arts Software GmbH', 'Infogrames', 'Insomniac Games',
       'Interchannel', 'Interchannel-Holon', 'Interplay',
       'Introversion Software', 'Irem Software Engineering',
       'Jack of All Games', 'Jaleco', 'JoWood Productions', 'Jorudan',
       'KID', 'KING Art Games', 'KOEI', 'Kadokawa Games',
       'Kadokawa Shoten', 'Kaga Create', 'Kalypso', 'Kalypso Media',
       'Kamui', 'Karin Entertainment', 'Kemco', 'Kids Station',
       'King Records', 'Klei Entertainment', 'Knowledge Adventure',
       'Koch Media', 'Koei Tecmo', 'Konami',
       'Konami Digital Entertainment', 'LEGO Media', 'Leadman Games',
       'League of Geeks', 'Legacy Interactive', 'Level 5', 'Licensed 4U',
       'Lighthouse Interactive', 'Lionhead Studios', 'Little Orbit',
       'LucasArts', 'M2', 'MLB Advanced Media', 'MLB.com', 'MTO',
       'MTV Games', 'Mad Catz', 'Magix', 'Majesco',
       'Majesco Entertainment', 'Mamba Games', 'Marvel Entertainment',
       'Marvelous', 'Marvelous Entertainment', 'Marvelous Interactive',
       'Masque Publishing', 'Mastertronic', 'Mastiff', 'Matatabi',
       'Maximum Family Games', 'Maximum Games', 'Maxis',
       'Media Entertainment', 'Media Factory', 'Media Works',
       'Mediascape', 'Mentor Interactive', 'Merge Games', 'Meridian4',
       'Merscom LLC', 'Metro 3D', 'Michaelsoft', 'Microids', 'Microsoft',
       'Microsoft Game Studios', 'Microsoft Studios',
       'Midas Interactive Entertainment', 'Midway Games', 'Milestone',
       'Milestone S.r.l', 'Milestone S.r.l.', 'Milkstone Studios',
       'Minato Station', 'Mindscape', 'Mirai Shounen', 'Modus Games',
       'Mojang', 'Monte Christo Multimedia', 'Moss',
       'Mud Duck Productions', 'Mumbo Jumbo', 'Myelin Media', 'NCSoft',
       'NEC Interchannel', 'NIS America', 'Namco', 'Namco Bandai',
       'Namco Bandai Games', 'Natsume', 'Navarre Corp', 'Naxat Soft',
       'Neko Entertainment', 'Neocore Games', 'NetRevo', 'NewKidCo',
       'Nexon', 'Nicalis', 'Nighthawk Interactive', 'Nihon Falcom Corp',
       'Nihon Falcom Corporation', 'Nintendo', 'Nippon Amuse',
       'Nippon Columbia', 'Nippon Cultural Broadcasting eXtend',
       'Nippon Ichi Software', 'Nitroplus', 'Nobilis', 'Nordic Games',
       'NovaLogic', 'Now Production', 'O-Games', 'O3 Entertainment',
       'Obsidian Entertainment', 'Office Create', 'Ongakukan',
       'Orbital Media, Inc.', 'Otomate', 'Otomate Idea Factory',
       'Outright Games', 'Oxygen Interactive', 'O~3 Entertainment',
       'P2 Games', 'PM Studios', 'PQube', 'Paon', 'Paon Corporation',
       'Paradox Development', 'Paradox Interactive',
       'Paramount Digital Entertainment', 'Perfect World Entertainment',
       'Perp Games', 'Perpetual', 'Phantagram', 'Phantom 8 Studio',
       'Phantom EFX', 'Piacci', 'Pinnacle', 'Pioneer LDC',
       'Planet Entertainment', 'PlayV', 'Playlogic Game Factory',
       'Playmore', 'Plenty', 'Polykid', 'PopCap Games', 'Princess Soft',
       'Prototype', 'Quinrose', 'Qute', 'RED Entertainment', 'RTL',
       'RailSimulator.com', 'Rain Games', 'Rare', 'Ravenscourt',
       'Ready at Dawn', 'Rebellion', 'Rebellion Developments',
       'Rebellion Games', 'Red Flagship', 'Red Mile Entertainment',
       'Red Storm Entertainment', 'RedOctane', 'Reef Entertainment',
       'Rejet', 'Revolution (Japan)', 'Revolution Software',
       'Rising Star', 'Rising Star Games', 'Rocket Company',
       'Rockstar Games', 'Rondomedia', 'Rooster Teeth Games', 'Russel',
       'SCS Software', 'SCi', 'SNK', 'SNK Playmore', 'SVG Distribution',
       'Sammy Corporation', 'Scholastic Inc.', 'Secret Stash Games',
       'Sega', "Shin'en", 'Shogakukan', 'Sierra Entertainment', 'Silky`s',
       'Simon & Schuster Interactive', 'Skybound Games',
       'Slitherine Software', 'Snail Games USA', 'Soedesco', 'Sold Out',
       'Sony Computer Entertainment',
       'Sony Computer Entertainment America',
       'Sony Interactive Entertainment', 'Sony Music Entertainment',
       'Sony Online Entertainment', 'SouthPeak Interactive', 'Spike',
       'Spike Chunsoft', 'Sprite', 'Square', 'SquareSoft',
       'Stainless Games', 'Stardock', 'Starfish', 'Sting',
       'Storm City Games', 'Strategy First', 'Studio Wildcard', 'Success',
       'Summitsoft', 'Sunrise Interactive', 'Sweets', 'System 3',
       'System 3 Arcade Software', 'System Soft', 'System Soft Alpha',
       'SystemSoft Alpha', 'TDK Core', 'TDK Mediactive', 'TGL', 'THQ',
       'THQ Nordic', 'TREVA Entertainment', 'Taito', 'Takara',
       'Takara Tomy', 'Take-Two Interactive', 'Takuyo', 'Team 17',
       'Team Meat', 'Team17 Digital Ltd', 'Team17 Software', 'Techland',
       'Technical Associates Inc', 'Tecmo', 'Tecmo Koei', 'Telegames',
       'Telltale Games', 'The Adventure Company', 'Thomas Happ Games',
       'Titus', 'Tivola', 'Tommo', 'Tomy Corporation',
       'TopWare Interactive', 'Touchstone', 'Tri Synergy',
       'Triangle Service', 'Trion Worlds', 'Tripwire Interactive',
       'Tru Blu Entertainment', 'Tryfirst', 'Tulip Games', 'Turbine Inc.',
       'Type-Moon', 'U&I Entertainment', 'UFO Interactive',
       'UIG Entertainment', 'Ubisoft', 'Unfinished Pixel',
       'Universal Interactive', 'Unknown', 'VU Games', 'Valcon Games',
       'ValuSoft', 'Valve', 'Victor Interactive', 'Views',
       'Villa Gorilla', 'Vir2L Studios', 'Virtual Play Games', 'Visco',
       'Viva Media', 'Vivendi Games', 'Warashi', 'Wargaming.net',
       'Warner Bros. Interactive',
       'Warner Bros. Interactive Entertainment',
       'WayForward Technologies', 'Wired Productions', 'Wolfgame',
       'Working Designs', 'XS Games', 'Xicat Interactive', 'Xseed Games',
       'Yamasa Entertainment', 'Yeti', "Yuke's", 'Zenrin',
       'Zoo Digital Publishing', 'Zoo Games', 'Zushi Games',
       'bitComposer Games', 'eGames', 'fonfun', 'honeybee', 'iMel',
       'iWin', 'imageepoch Inc.', 'inXile Entertainment', 'responDESIGN'];
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
        .force("charge", d3.forceManyBody().strength(-8))
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