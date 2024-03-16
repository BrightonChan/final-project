<script>
  // Write your JS here, or import other files
  //import { base } from '$app/paths';
  import { onMount } from 'svelte';
  //import pkg from 'lodash';
  //const { debounce } = pkg;
  import * as d3 from 'd3';
  import { fade } from 'svelte/transition';
  
  let hovered = -1;
  let data = '';

  onMount(async () => {
      const response = await fetch('rookie.csv');
      const csvData = await response.text();
      data = d3.csvParse(csvData, (d) => ({
      player: d.Player,
      Steals: +d.STL,
      Blocks: +d.BLK,
    }));
    console.log(data);
    drawChart();
  });

  onMount(async () => {
     const response = await fetch('wemby.csv');
     const csvData = await response.text();
     data = d3.csvParse(csvData, (d) => ({
     PTS: +d.PTS,
     REB: +d.REB,
     AST: +d.AST,
     STL: +d.STL,
     BLK: +d.BLK,
     MIN: +d.MIN,
     FG_PCT: +d.FG_PCT,
     FG3_PCT: +d.FG3_PCT,
     MONTH: +d.MONTH,
   }));


   console.log(data);
   drawLineChart();
 });


	
	function drawChart() {
    var margin = {top: 10, right: 0, bottom: 75, left: 150},
    width = 1000 - margin.left - margin.right,
    height = 600 - margin.top - margin.bottom;

    const svg = d3.select('#chart').append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform",
          "translate(" + margin.left + "," + margin.top + ")");

          var subgroups = ["Steals","Blocks"];

// List of groups = species here = value of the first column called group -> I show them on the X axis
var groups = d3.map(data, function(d){return(d.player)})
// Add X axis

var x = d3.scaleLinear()
    .domain([0, 6])  // Adjust the domain as needed
    .range([0, width]);

// Append the x-axis
svg.append("g")
  .attr("transform", "translate(0," + height + ")")  // Move the x-axis to the bottom
  .call(d3.axisBottom(x).tickSizeOuter(0));
  
svg.append("text") // Add x-axis label
    .attr("x", width - 450)
    .attr("y", height + 50) // Adjust position as needed
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .text("Steals and Blocks");

// Define the y-axis scale
var y = d3.scaleBand()
    .domain(groups)
    .range([height, 0])  // Reverse the range to flip the axis
    .padding([0.2]);

// Append the y-axis
svg.append("g")
  .call(d3.axisLeft(y));

svg.append("text") // Add y-axis label
    .attr("transform", "rotate(-90)")
    .attr("x", -250)
    .attr("y", -150) // Adjust position as needed
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .text("Rookies"); // Label text

// color palette = one color per subgroup
var color = d3.scaleOrdinal()
  .domain(subgroups)
  .range(['#e41a1c','#377eb8','#4daf4a'])

//stack the data? --> stack per subgroup
var stackedData = d3.stack()
  .keys(subgroups)
  (data)
// console.log(stackedData);
// add legend
var legend = svg.selectAll(".legend")
      .data(subgroups)
      .enter().append("g")
        .attr("class", "legend")
        .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; });
      
    legend.append("rect")
      .attr("x", width - 21)
      .attr("width", 20)
      .attr("height", 18)
      .style("fill", color);

    // Legend text
    legend.append("text")
      .attr("x", width - 24)
      .attr("y", 9)
      .attr("dy", ".35em")
      .style("text-anchor", "end")
      .text(function(d) { return d; })
      .style("fill", function(d){ return "black"});      ;
      
      const tooltip = d3.select("#chart")
    .append("div")
    .style("opacity", 0)
    .attr("class", "tooltip")
    .style("background-color", "white")
    .style("border", "solid")
    .style("border-width", "1px")
    .style("border-radius", "5px")
    .style("padding", "10px")
    
const mouseover = function(event, d) {
    const playerName = d.data.player;
    const subgroupName = d3.select(this.parentNode).datum().key;
    const stealValue = d.data.Steals;
    const blockValue = d.data.Blocks;
    const total = (stealValue + blockValue).toFixed(1);

    
    const tooltip = document.getElementById('tooltip');
    tooltip.innerHTML = `Player: ${playerName }`+ 
                        `<br>Steals: ${stealValue}` + 
                        `<br>Blocks: ${blockValue}`+ 
                        `<br> Total: ${total}`;
        
    // Set tooltip position relative to the mouse cursor
    tooltip.style.left = `${event.pageX + 230}px`; // Adjust offset as needed
    tooltip.style.top = `${event.pageY -50}px`; // Adjust offset as needed

    tooltip.style.opacity='1';
    tooltip.style.display = 'inline-block';
    
};
//change the pos
const mousemove = function(event, d) {
    tooltip.style("left",(event.x)/2+"px")
           .style("top",(event.y)/2+"px")
  }
  const mouseleave = function(event, d) {
    tooltip.style("opacity", 0)
  }
// Show the bars
svg.append("g")
  .selectAll("g")
  // Enter in the stack data = loop key per key = group per group
  .data(stackedData)
  .enter().append("g")
    .attr("fill", function(d) { return color(d.key); })
    .selectAll("rect")
    // enter a second time = loop subgroup per subgroup to add all rectangles
    .data(function(d) { return d; })
    .enter().append("rect")
      .attr("y", function(d) { return y(d.data.player); })
      .attr("x", function(d) { return x(d[0]); })
      .attr("width", function(d) { return x(d[1]) - x(d[0]); })
      .attr("height",y.bandwidth())
    .on("mouseover", mouseover)
    .on("mousemove", mousemove)
    .on("mouseleave", mouseleave)

  
    }

  let svg;
  let showPlayerInfo = true;
  let selectedPlayer = "Victor Wembanyama";
  let width = 0;
  let height = 0;
  let players = ["Victor Wembanyama", "Paolo Banchero", "Scottie Barnes", "Lamelo Ball", "Ja Morant", "Luka Doncic", "Ben Simmons", "Malcolm Brogdon", "Karl-Anthony Towns", "Andrew Wiggins", "Michael Carter-Williams", "Damian Lillard", "Kyrie Irving", "Blake Griffin", "Tyreke Evans", "Derrick Rose", "Kevin Durant", "Brandon Roy", "Chris Paul", "Emeka Okafor", "Lebron James"];
  
  let averages = {"Above the Break 3": 0.286380952,
                    "In The Paint (Non-RA)": 0.391761904,
                    "Left Corner 3": 0.273380952,
                    "Mid-Range": 0.377714285,
                    "Restricted Area": 0.604619047,
                    "Right Corner 3": 0.351857142}
  let playerShootingData;

  onMount(() => {
    const container = document.getElementById('container');
    width = container.offsetWidth;
    height = container.offsetHeight;
    svg = d3.select('#overlay');

    fetch("player_stats.json")
      .then(response => response.json())
      .then(data => {
        playerShootingData = data;
        renderSVG();
      })
      .catch(error => console.error('Error loading player shooting data:', error));
    });

    // Handles resizing of window
    function handleResize() {
        const container = document.getElementById('container');
        width = container.offsetWidth;
        height = container.offsetHeight;
        renderSVG();
    }

    // Displays tooltip on hover
    function showTooltip(data, event) {
        const tooltip = document.getElementById('tooltip');
        tooltip.innerHTML = `<b>${data.BASIC_ZONE}</b>:<br>` + 
                            `FG Made: ${data.sum}<br>` +
                            `FG Attempted: ${data.count}<br>` +
                            `FG%: ${String(Math.round(data.mean * 100 * 100) / 100) + '%'}<br>` +
                            `Rookies Average FG%: ${String(Math.round(averages[data.BASIC_ZONE]*100)) + '%'}`;
        
        // Set tooltip position relative to the mouse cursor
        tooltip.style.left = `${event.pageX + 10}px`; // Adjust offset as needed
        tooltip.style.top = `${event.pageY + 10}px`; // Adjust offset as needed

        tooltip.style.opacity='1';
        tooltip.style.display = 'block';
    }  

    function showvideo(data, event) {
        const videoPlayer = document.getElementById('videoPlayer');
        console.log(videoPlayer);
        // Check if zoneData contains valid video information
        if (data && data.videoUrl) {
        // Set the video source to the URL from zoneData
            videoPlayer.src = data.videoUrl;

        // Play the video
            videoPlayer.play();
        } else {
        // Handle the case where there is no valid video information in zoneData
            console.error('Invalid or missing video information for the selected zone.');
        }
    }


    function hideTooltip() {
        const tooltip = document.getElementById('tooltip');
        tooltip.style.display = 'none';
        tooltip.style.opacity = 0; // Reset opacity for the next time it's shown
    }
    let leftcornerregion = { x: 0, y: 40.2, width: 6, height: 28.5};
    let rightcornerregion = { x: 93.9, y: 40.2, width: 6.5, height: 28.5};
    let paintregion = { x: 33.9, y: 56, width: 32, height: 44.2 };
    let restrictedregion = { x: 49.9, y: 14.5 };
    let restrictedregion_rec = { x: 42, y: 14.55, width: 15.9, height: 2.4 };
    let middle_outer = {x: 0, y: 100, width: 100, height: 44.2}
    let left_outer = {x: 0, y: 55.8, width: 35, height: 15.8}
    let right_outer = {x: 65.9, y: 55.8, width: 35, height: 15.8}
    let mid_left = {x: 6, y: 40.2, width: 28, height: 28.5}
    let mid_right = {x: 66, y: 40.2, width: 27.8, height: 28.5}
    let mid_middle = { x: 50, y: 14};

    function renderSVG() {
        if (!svg) {
            console.error('SVG container not found');
            return;
        }
        console.log('Rendering SVG');

        svg.selectAll('*').remove();

        const xScale = d3.scaleLinear().domain([0, 100]).range([0, width]);
        const yScale = d3.scaleLinear().domain([0, 100]).range([height, 0]);

        const xAxis = d3.axisBottom(xScale);
        const yAxis = d3.axisLeft(yScale);
        
        svg.attr('width', width).attr('height', height);

        svg.append('g').attr('transform', `translate(0,${height})`).call(xAxis);
        svg.append('g').call(yAxis);



        // GENERAL 3PT AREA SVG
        let aboveTheBreak3Data = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Above the Break 3");
        aboveTheBreak3Data.forEach(data => {
            let tooltipText = `${data.BASIC_ZONE}: ${data.mean}`;
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#FF0000';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.04) {
                fillcolor = '#FF4500';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#FFA500';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#FFFF00';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#9ACD32';
            } else if (data.mean - averages[data.BASIC_ZONE] < .04) {
                fillcolor = '#008000';
            } else {
                fillcolor = '#006400';
            }
            svg.append('rect')
                .attr('x', xScale(middle_outer.x))
                .attr('y', yScale(middle_outer.y))
                .attr('width', xScale(middle_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(middle_outer.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            svg.append('rect')
                .attr('x', xScale(left_outer.x))
                .attr('y', yScale(left_outer.y) - 2)
                .attr('width', xScale(left_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(left_outer.height) + 4)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            svg.append('rect')
                .attr('x', xScale(right_outer.x))
                .attr('y', yScale(right_outer.y) - 2)
                .attr('width', xScale(right_outer.width) - xScale(0))
                .attr('height', yScale(0) - yScale(right_outer.height) + 4)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // MID RANGE AREA SVG
        let mid_range = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Mid-Range");
        mid_range.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#FF0000';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.04) {
                fillcolor = '#FF4500';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#FFA500';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#FFFF00';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#9ACD32';
            } else if (data.mean - averages[data.BASIC_ZONE] < .04) {
                fillcolor = '#008000';
            } else {
                fillcolor = '#006400';
            }
            svg.append('rect')
                .attr('x', xScale(mid_left.x))
                .attr('y', yScale(mid_left.y))
                .attr('width', xScale(mid_left.width) - xScale(0))
                .attr('height', yScale(0) - yScale(mid_left.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
            let threePt_radius = Math.min(width, height) / 2 * 1.39;
            let threePtStartAngle = Math.PI;
            let threePtEndAngle = 0;

            // Calculate the start and end points of the arc
            let threePtStartX = threePt_radius * Math.cos(threePtStartAngle);
            let threePtStartY = threePt_radius * Math.sin(threePtStartAngle);
            let threePtEndX = threePt_radius * Math.cos(threePtEndAngle);
            let threePtEndY = threePt_radius * Math.sin(threePtEndAngle);

            // Create the arc path manually
            let threePtArcPath = `M ${threePtStartX} ${threePtStartY} A ${threePt_radius} ${threePt_radius} 0 0 1 ${threePtEndX} ${threePtEndY}`;

            svg.append('path')
                .attr('transform', `translate(${xScale(mid_middle.x)}, ${yScale(mid_middle.y)})`)
                .attr('d', threePtArcPath)
                .attr('fill', 'none')
                .attr('stroke', 'black')
                .attr('stroke-width', '10');
    
            svg.append('rect')
                .attr('x', xScale(mid_right.x))
                .attr('y', yScale(mid_right.y))
                .attr('width', xScale(mid_right.width) - xScale(0))
                .attr('height', yScale(0) - yScale(mid_right.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            let second_radius = Math.min(width, height) / 2 * 1.40;
            const darc = d3.arc()
                .innerRadius(0)
                .outerRadius(second_radius)
                .startAngle(-Math.PI / 2)
                .endAngle(Math.PI / 2);

            svg.append('path')
                .attr('transform', `translate(${xScale(mid_middle.x)}, ${yScale(mid_middle.y)})`)
                .attr('d', darc)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // RIGHT CORNER 3PT AREA SVG
        let left_cornery = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Left Corner 3");
        left_cornery.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#FF0000';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.04) {
                fillcolor = '#FF4500';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#FFA500';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#FFFF00';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#9ACD32';
            } else if (data.mean - averages[data.BASIC_ZONE] < .04) {
                fillcolor = '#008000';
            } else {
                fillcolor = '#006400';
            }
            svg.append('rect')
                .attr('x', xScale(rightcornerregion.x))
                .attr('y', yScale(rightcornerregion.y))
                .attr('width', xScale(rightcornerregion.width) - xScale(0))
                .attr('height', yScale(0) - yScale(rightcornerregion.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // LEFT CORNER 3PT AREA SVG
        let right_cornery = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Right Corner 3");
        right_cornery.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#FF0000';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.04) {
                fillcolor = '#FF4500';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#FFA500';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#FFFF00';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#9ACD32';
            } else if (data.mean - averages[data.BASIC_ZONE] < .04) {
                fillcolor = '#008000';
            } else {
                fillcolor = '#006400';
            }
            svg.append('rect')
                .attr('x', xScale(leftcornerregion.x))
                .attr('y', yScale(leftcornerregion.y))
                .attr('width', xScale(leftcornerregion.width) - xScale(0))
                .attr('height', yScale(0) - yScale(leftcornerregion.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // IN THE PAINT (NON-RA) AREA SVG
        let dapaint = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "In The Paint (Non-RA)");
        dapaint.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#FF0000';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.04) {
                fillcolor = '#FF4500';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#FFA500';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#FFFF00';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#9ACD32';
            } else if (data.mean - averages[data.BASIC_ZONE] < .04) {
                fillcolor = '#008000';
            } else {
                fillcolor = '#006400';
            }
            svg.append('circle')
                .attr('cx', xScale(paintregion.x + 16))
                .attr('cy', yScale(paintregion.y))
                .attr('r', xScale(10.8))
                .attr('stroke', 'black')
                .attr('stroke-width', 2)
                .attr('fill', 'none');
            svg.append('rect')
                .attr('x', xScale(paintregion.x))
                .attr('y', yScale(paintregion.y))
                .attr('width', xScale(paintregion.width) - xScale(0))
                .attr('height', yScale(0) - yScale(paintregion.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });



        // RESTRICTED AREA SVG
        let dara = playerShootingData[selectedPlayer].filter(data => data.BASIC_ZONE === "Restricted Area");
        dara.forEach(data => {
            let fillcolor = 'black'
            if (data.mean - averages[data.BASIC_ZONE] < -.06) {
                fillcolor = '#FF0000';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.04) {
                fillcolor = '#FF4500';
            } else if (data.mean - averages[data.BASIC_ZONE] < -.02) {
                fillcolor = '#FFA500';
            } else if (data.mean - averages[data.BASIC_ZONE] < 0) {
                fillcolor = '#FFFF00';
            } else if (data.mean - averages[data.BASIC_ZONE] < .02) {
                fillcolor = '#9ACD32';
            } else if (data.mean - averages[data.BASIC_ZONE] < .04) {
                fillcolor = '#008000';
            } else {
                fillcolor = '#006400';
            }
            let radius = Math.min(width, height) / 2 * .235;
            const arc = d3.arc()
                .innerRadius(0)
                .outerRadius(radius)
                .startAngle(-Math.PI / 2)
                .endAngle(Math.PI / 2);

            svg.append('path')
                .attr('transform', `translate(${xScale(restrictedregion.x)}, ${yScale(restrictedregion.y)})`)
                .attr('d', arc)
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);

            svg.append('rect')
                .attr('x', xScale(restrictedregion_rec.x))
                .attr('y', yScale(restrictedregion_rec.y))
                .attr('width', xScale(restrictedregion_rec.width) - xScale(0))
                .attr('height', yScale(0) - yScale(restrictedregion_rec.height))
                .attr('fill', fillcolor)
                .attr('fill-opacity', '1')
                .attr('BASIC_ZONE', data.BASIC_ZONE)
                .on('click', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showvideo(zoneData, event);
                })
                .on('mouseover', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Pass the event object
                })
                .on('mousemove', (event) => {
                    const zone = event.target.getAttribute('BASIC_ZONE');
                    const zoneData = playerShootingData[selectedPlayer].find(d => d.BASIC_ZONE === zone);
                    showTooltip(zoneData, event); // Update tooltip position on mouse move within the area
                })
                .on('mouseout', hideTooltip);
        });


        
        // COVER BOTTOM OF NBA COURT IMAGE
        let bottomcornerregion = { x: 0, y:12, width: 100, height: 20};

        svg.append('rect')
            .attr('x', xScale(bottomcornerregion.x))
            .attr('y', yScale(bottomcornerregion.y))
            .attr('width', xScale(bottomcornerregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(bottomcornerregion.height))
            .attr('fill', 'white')
            .attr('fill-opacity', '1');



        // LINE DRAWINGS ON SVG
        svg.append('rect')
            .attr('x', xScale(paintregion.x))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(paintregion.width) - xScale(0))
            .attr('height', yScale(0) - yScale(paintregion.height)-2)
            .attr('fill', 'none')
            .attr('fill-opacity', '1')
            .attr('stroke', 'black')
            .attr('stroke-width', '3');

        let outline_radius = Math.min(width, height) / 2 * .235;
        let startAngle = Math.PI;
        let endAngle = 0;
        let startX = outline_radius * Math.cos(startAngle);
        let startY = outline_radius * Math.sin(startAngle);
        let endX = outline_radius * Math.cos(endAngle);
        let endY = outline_radius * Math.sin(endAngle);
        let arcPath = `M ${startX} ${startY} A ${outline_radius} ${outline_radius} 0 0 1 ${endX} ${endY}`;

        svg.append('path')
            .attr('transform', `translate(${xScale(restrictedregion.x)}, ${yScale(restrictedregion.y)})`)
            .attr('d', arcPath)
            .attr('fill', 'none')
            .attr('stroke', 'black')
            .attr('stroke-width', '3');

        svg.append('rect')
            .attr('x', xScale(leftcornerregion.x + 5.3))
            .attr('y', yScale(leftcornerregion.y))
            .attr('width', xScale(0.5))
            .attr('height', xScale(19))
            .attr('fill', 'black')
            .attr('stroke', 'black');

        svg.append('rect')
            .attr('x', xScale(rightcornerregion.x + 0.17))
            .attr('y', yScale(rightcornerregion.y))
            .attr('width', xScale(0.5))
            .attr('height', xScale(19))
            .attr('fill', 'black')
            .attr('stroke', 'black');
        svg.append('rect')
            .attr('x', xScale(paintregion.x + 5.1))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(0.1))
            .attr('height', xScale(29.6))
            .attr('fill', 'black')
            .attr('stroke', 'black');

        svg.append('rect')
            .attr('x', xScale(paintregion.x + 26.8))
            .attr('y', yScale(paintregion.y))
            .attr('width', xScale(0.1))
            .attr('height', xScale(29.6))
            .attr('fill', 'black')
            .attr('stroke', 'black');
        svg.append('circle')
            .attr('cx', xScale(paintregion.x + 16))
            .attr('cy', yScale(paintregion.y))
            .attr('r', xScale(10.8))
            .attr('stroke', 'black')
            .attr('stroke-width', 2)
            .attr('fill', 'none')
            .style("stroke-dasharray", ("3, 5"));
        svg.append('circle')
            .attr('cx', xScale(paintregion.x + 16))
            .attr('cy', yScale(paintregion.y - 40.5))
            .attr('r', xScale(1.8))
            .attr('stroke', 'orange')
            .attr('stroke-width', 1)
            .attr('fill', 'none');
    }

    // Handle dropdown change event
    function handleDropdownChange(event) {
        // Get the selected option value
        const selectedOption = event.target.value;
        // Update selected player
        selectedPlayer = selectedOption;
        // Get the corresponding photo filename from the mapping object
        renderSVG();
    }


    function drawLineChart() {
   var margin = {top: 60, right: 0, bottom: 100, left: 100},
   width = 800 - margin.left - margin.right,
   height = 500 - margin.top - margin.bottom;


   const svg = d3.select('#linechart').append("svg")
   .attr("width", width + margin.left + margin.right)
   .attr("height", height + margin.top + margin.bottom)
 .append("g")
   .attr("transform",
         "translate(" + margin.left + "," + margin.top + ")");


        
   var subgroups = ["PTS","REB","AST","STL","BLK","MIN","FG_PCT","FG3_PCT"];


// List of groups = species here = value of the first column called group -> I show them on the X axis
// var groups = d3.map(data, function(d){return(d.player)})
   d3.select("#selectButton")
     .selectAll('myOptions')
     .data(subgroups)
     .enter()
     .append('option')
     .text(function (d) { return d; }) // text showed in the menu
     .attr("value", function (d) { return d; }) // corresponding value returned by the button


// add title
svg.append("text")
       .attr("x", (width / 2))            
       .attr("y", 0)
       .attr("text-anchor", "middle") 
       .style("font-size", "20px")
       .text("Victor Wemby's Performance Trend: A Month-by-Month Analysis");


// Add X axis
const x = d3.scaleBand()
   .domain(data.map(d => d.MONTH)) // Assuming MONTH is represented as numeric values (1, 2, 3, ...)
   .range([0, width]) // Adjust the range as needed
// Adjust the padding between bars if needed


// Append the x-axis
svg.append("g")
   .attr("transform", "translate(0," + height + ")")
   .call(d3.axisBottom(x)
       .tickFormat((d) => {
           const monthNames = ["2023/10", "2023/11", "2023/12", "2024/1", "2024/2"]; // Array of month abbreviations
           return monthNames[d - 1]; // Subtract 1 because months are zero-indexed in arrays
       })
   )
   .selectAll("text")
   .attr("transform", "rotate(-45)")
   .style("text-anchor", "end")
   .style("font-size",16)
   .style("fill", "#69a3b2");


// Define the y-axis scale
var y = d3.scaleLinear()
   .domain([0,30])
   .range([height, 0]);  // Reverse the range to flip the axis


// Append the y-axis
var yAxis = svg.append("g")
 .call(d3.axisLeft(y));


// color palette = one color per subgroup
var myColor = d3.scaleOrdinal()
     .domain(subgroups)
     .range(d3.schemeSet2);


const line = svg
     .append('g')
     .append("path")
       .datum(data)
       .attr("d", d3.line()
         .x(function(d) {return x(+d.MONTH)+90})
         .y(function(d) {return y(+d.PTS)})
       )
       .attr("stroke", function(d){ return myColor("valueA") })
       .style("stroke-width", 4)
       .style("fill", "none");
  
       // x,y axis label
   svg.append("text")
     .attr("class", "x label")
       .attr("text-anchor", "end")
       .attr("x", width/2)
       .attr("y", height + margin.bottom)
       .attr("fill", "#5D6971")
       .attr("font-size",16)
     .text("Date");
  
   var ylabel=svg.append("text")
     .attr("class", "y label")
     .attr("transform", "rotate(-90)")
       .attr("y", 0 -margin.left)
       .attr("x",0 - (height / 2))
       .attr("dy", "1em")
       .style("text-anchor", "middle")
       .attr("fill", "#5D6971")
       .attr("font-size",16)
     .text("Average Points by Month");




   // A function that update the chart
function update(selectedGroup) {


     // Create new data with the selection?
     const dataFilter = data.map(function(d){return {MONTH: d.MONTH, value:d[selectedGroup]} })
     var val=d3.max(dataFilter.map(d=>d.value)) *1.4
     const mappingTable = {
       "PTS": "Points",
       "REB": "Rebounds",
       "AST": "Assists",
       "STL": "Steals",
       "BLK": "Blocks",
       "MIN": "Minutes",
       "FG_PCT": "Field Goal Percentage",
       "FG3_PCT": "Three-Point Field Goal Percentage"
     };
     function mapColumnName(columnName) {
         return mappingTable[columnName]
     }
     var newLabel=mapColumnName(selectedGroup)
     //y = d3.scaleLinear()
     //.domain([0,val])
     //.range([height, 0]);
    
     y.domain([0,val])
     yAxis.transition()
     .duration(1000)
     .call(d3.axisLeft(y))
     //svg.append("g")
       //.call(d3.axisLeft(y));
     // Give these new data to update line
    
     ylabel.text("Average " + newLabel + " by Month");
     line
         .datum(dataFilter)
         .transition()
         .duration(1000)
         .attr("d", d3.line()
           .x(function(d) { return x(+d.MONTH)+90 })
           .y(function(d) { return y(+d.value) })
         )
         .attr("stroke", function(d){ return myColor(selectedGroup) })
   }
  


   // When the button is changed, run the updateChart function
   d3.select("#selectButton").on("change", function(event, d) {
       // recover the option that has been chosen
       var selectedOption = d3.select(this).property("value")
       // run the updateChart function with this selected option
       update(selectedOption)  
   })
  }

  function scrollToOffense() {
    // Get a reference to the target div
    const targetDiv = document.getElementById('Offense');
    // Scroll to the target div
    targetDiv.scrollIntoView({ behavior: 'smooth' });
  }

  function scrollToDefense() {
    // Get a reference to the target div
    const targetDiv = document.getElementById('defense');
    // Scroll to the target div
    targetDiv.scrollIntoView({ behavior: 'smooth' });
  }

  function changeImageSize() {
        var value = document.getElementById("myRange").value;
        var curryImg = document.getElementById("curry-img");
        curryImg.style.width = value*2.1 + "px";
    }
</script>

<main>
    <div class="page" id="first" >


    </div>
     <div class="page2">
      <p class="topic"><b>Why is Wembanyama so Hyped Up?</b></p>
      <p class="left_txt">- 2023 First Draft Pick</p>
      <p class="left_txt">- More than 6 teams tanking for him (losing on purpose)</p>
      <p class="left_txt">- Listed Height: 7 foot 3</p>
      <p class="left_txt">- Wingspan: 8 foot</p>
      <p class="left_txt">- Elected MVP, Best Young Player, Best Defender</p>
      <p class="left_txt">- Top Scorer and Best Blocker in the French League</p>
      <p class="left_txt">- One of the most hyped rookies since LeBron James</p>
    </div>
    <div transition:fade>
        <h1>Just How Tall is Wembanyama?</h1>
        <h2>Compare Your Height with Wemby!!</h2>
    </div>
    <div id="ill">
        <div class="input-wrapper">
        <label for="height1">How tall are you?</label>
        <input type="range" min="130" max="240" value="188" class="slider" id="myRange" oninput="rangeValue.innerText = this.value" on:input={changeImageSize}>
        <div class="value-container">
            <p id="rangeValue">188</p>
            <p>cm</p>
        </div>
    </div>
    <div class="image-wrapper">
        <img src={'wembyimg.jpeg'} alt="Wembanyama" id="wembanyama-img">
    </div>
    <div class="image-wrapper">
        <img src={'curry.png'} alt="Curry" id="curry-img">
    </div>
    </div>
    <h1>Now, Onto why he's one of the best rookies ever!</h1>
    <h2>Click on me!</h2>
    <div id="buttons">
        <button on:click={scrollToOffense}>Offense</button>
        <button on:click={scrollToDefense}>Defense</button>
    </div>
    <h2 id="defense">Defensive Impact</h2>
    <h3 id="q1">How is Wembanyama's defensive performance compared to other rookies?</h3>
    <div id="chart"></div>
    <h2 id="Offense">Offensive Impact</h2>
    <h3 id="q1">How effective is Wembanyama shooting from the field compared to past rookie of the years?</h3>
    <h3 id="clicking">(Try to click on the zones !)</h3>
    <div id="container">
        <img src={'nbacourt.jpg'} alt="Basketball Court" id="bball">
        <svg id="overlay"></svg>
        {#if showPlayerInfo}
            <div id="player-info">

            <!-- Player information -->
            <h2>{selectedPlayer}</h2>

            <!-- Dropdown menu for changing players -->
            <label for="playerDropdown">Change Player:</label>
            <select id="playerDropdown" on:change={handleDropdownChange}>
                {#each players as player}
                    <option value={player}>{player}</option>
                {/each}
            </select>
            <video id="videoPlayer" controls></video>
        </div>
    {/if}
    </div>
    <div id="tooltip" style="position: absolute; opacity: 1; pointer-events: none; transition: opacity 0.2s;"></div>

<div id="legend">
<div id="gradient-legend">
    <div class="gradient-bar"></div>
    <div class="percentage-scale">
        <span>-6%</span>
        <span>-4%</span>
        <span>-2%</span>
        <span>0%</span>
        <span>+2%</span>
        <span>+4%</span>
        <span>+6%</span>
        <span>NA</span>
    </div>
</div>
<div class="frequency-scale">
    <span>Field Goal % of Rookie vs. Rookie of the Year Averages</span>
</div>
</div>
<select id="selectButton"></select>
<div id="linechart"></div> 
</main>

<style>

div {
    background-image: url('wemby2.jpg');
   background:burlywood;
 }
 .input-wrapper{
    padding-left: 30%;
 }
    .p{
   position: relative;
 }
  .page{
   padding-top: 40%;
   padding-bottom: 30%;
   padding-right: 40%;
   padding-left: 40%;
 }
 .page::before {
   width: 600px; /* Set width to cover the entire .page element */
   height:14%;
   content: 'Is All the Hype Around Victor Wembanyama Legit?'; /* Text content */
   position: absolute;
   top: 70%; /* Adjust vertical positioning as needed */
   left: 50%; /* Adjust horizontal positioning as needed */
   transform: translate(-50%, -50%); /* Center the text */
   text-align: center;
   text-anchor: middle;
   display:flex;
   font-size: 40px;
   color: #333;
   font-family: Arial, sans-serif;
   background-color: rgba(255, 255, 255, 0.5); /* Semi-transparent white color layer */
   }
 #first{
   background-image:url('wemby2.jpg');
   background-size: cover;
   background-position: center;
   background-repeat: no-repeat;
   height: 50%;
   width: 20%;
 }
 .page2{
   padding-right: 40%;
   background-color: lightgrey;
 }
 .topic {
   width: 1200px;
   height: 100px;
   top: 10;
   left: 50;
   font-size: 60px;
   text-align: center;
   margin-left: 5%;
   margin-right: 5%;
 }
 .left_txt{
   width: 750px;
   height: 60px;
   top: 0;
   left: 0;
  
   font-size: 35px;
   margin-left: 28%;
   color: orangered;
 }

    main {
    background-color: burlywood;
    }   

    #q1 {
        text-align: center;
        padding: 4px;
    }
    #chart {
        width: 80%; /* Set the width of the chart to 80% of its container's width */
        height: 650px; /* Set the height of the chart to 400 pixels */
        margin: 0 auto; /* Center the chart horizontally */
        text-align: center;
        padding: 20px;
    }
    .value-container {
            display: flex;
            align-items: baseline;
        }
    #container {
        animation: fadeIn 1s ease-out;
        position: relative;
        width: 50%;
        margin-left: 8%;
        margin-bottom: 30px;
    }

    #clicking {
        margin-left: 100px;
    }
    h1 {
        animation: fadeInright 1s ease-out;
        text-align: center;
        font-family: 'Playfair Display'; 
        font-size: 45px;
        margin-top: 10px;
    }

    h2 {
        animation: fadeIn 1s ease-out;
        text-align: center;
        margin-top: 5%;
        margin-bottom: 0px;
        padding-top: 0px;
    }

    h4 {
        animation: fadeInright 1s ease-out;
        text-align: center;
        font-family: 'PT Sans', sans-serif;
        margin-left: 5%;
        margin-right: 5%;
        margin-top: 0%;
        margin-bottom: 2%;
        padding-top: 0px;
    }

    p {
        animation: fadeIn 1.5s ease-out;
        text-align:center;
        padding: 1% 10%;
        margin-top: 0px;
    }

    hr {
        border: none; /* Removes the default border */
        height: 1px; /* Sets the height of the line */
        background-color: grey; /* Sets the color of the line */
        margin-top: 5rem; /* Adds space above the line */
        margin-bottom: 0rem; /* Adds space below the line */
    }

    .image-wrapper {
            width: 100%; /* Adjust as needed */
            /*margin-right: 40px;*/
            position: relative;
    }

    #myRange {
        width: 50%;
        height: 20px;
        margin: 10px;
    }
    #wembanyama-img {
        padding-left: 27%;
            width: 40%;
            /*margin-right: 40px;*/
        }
    
    #curry-img {
        padding-left: 27%;
        width: 28.3%;
        position: absolute;
        bottom: 10px;
        left: 350px; /* Adjust positioning as needed*/
        z-index: 1; /* Ensure this image appears above the other */
    }

    .input-wrapper {
            display: flex;
            flex-direction: column;
            align-items: flex-start;
            margin-left: 20px; /* Adjust as needed for spacing between image and input */
            position: relative;
        }
    #bball {
        display: block;
        width: 100%;
        height: auto;
        left: 500px;
        z-index:4;
    }


    video {
        display: center;
        width: 320%;
        height: auto;
        right: 50px;
        z-index:4;
    }


    #overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 2;
    }

    #player-info {
        position: absolute;
        top: 0;
        left: 105%;
        text-align: center;
        color: black;
        font-family: Arial, sans-serif;
    }

    #player-image {
        width: 263px;
        height: 192px;
        margin-left: auto;
        margin-right: auto;
        margin-bottom: 20px;
    }

    #tooltip {
        background: white;
        border: 1px solid #ccc;
        padding: 10px;
        border-radius: 5px;
        display: none; /* Hidden by default */
        z-index: 3;
        position:absolute;
        transition: opacity 0.2s;
    }

    #legend {
        animation: fadeIn 1s ease-out;
        font-family: Arial, sans-serif;
        display: flex;
        flex-direction: column;
        align-items: left;
        margin-top: 20px;
        margin-left: 8%
    }
    #selectButton {
        font-size: 10px;
        color: #200f8d;
        background-color: #fff;
        border: 5px solid #ccc;
        padding: 3px;
    }


    #gradient-legend {
        position: relative;
        margin-bottom: 10px;
    }

    label {
        font-weight: bold;
    }

    select {
        background-color: rgb(225, 225, 225);
        color: black; /* Setting the text color to white for better contrast */
        border: 1.3px solid #ccc; /* Optional: adds a border */
        padding: 0.4em; /* Optional: adds some padding inside the dropdown */
        border-radius: 8px; /* Optional: rounds the corners of the dropdown */
        margin-top: 2%;
        text-align: center;
    }

    .gradient-bar {
        width: 240px; /* Increased width to accommodate NA section */
        height: 20px;
        background: linear-gradient(to right, 
                                    #FF0000 0%, #FF0000 12.5%, /* Significantly below average */
                                    #FF4500 12.5%, #FF4500 25%,
                                    #FFA500 25%, #FFA500 37.5%, /* Below average */
                                    #FFFF00 37.5%, #FFFF00 50%, /* Slightly above average */
                                    #9ACD32 50%, #9ACD32 62.5%, /* Above average */
                                    #008000 62.5%, #008000 75%,
                                    #006400 75%, #006400 87.5%, /* Significantly above average */
                                    #a3a2af 87.5%, #a3a2af 100% /* Not applicable */
                                    );
        border: 1px solid #ccc;
    }

    .percentage-scale {
        display: flex;
        justify-content: space-between;
        width: 240px; /* Increased width to match gradient bar */
    }

    .frequency-scale span {
        margin-bottom: 5px;
    }
    
    .fade-transition {
    transition: opacity 0.5s;
    }

    #buttons {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 5vh; /* Center vertically */
    padding-top: 20px;
    }

    button {
        padding: 10px 20px; /* Increase padding to make buttons larger */
        font-size: 18px; /* Increase font size */
        margin: 10px; /* Add some margin between buttons */
    }
    
</style>


