<script>
  // Write your JS here, or import other files
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
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

	
	function drawChart() {
    var margin = {top: 10, right: 0, bottom: 20, left: 150},
    width = 500 - margin.left - margin.right,
    height = 400 - margin.top - margin.bottom;

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

// Define the y-axis scale
var y = d3.scaleBand()
    .domain(groups)
    .range([height, 0])  // Reverse the range to flip the axis
    .padding([0.2]);

// Append the y-axis
svg.append("g")
  .call(d3.axisLeft(y));

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
    const total =(stealValue+blockValue).toFixed(1)
    tooltip
        .html(playerName+"<br>Steals: " + stealValue + "<br>Blocks: " + blockValue +"<br> Total:"+total)
        .style("opacity", 1)

  }
//change the pos
const mousemove = function(event, d) {
    tooltip.style("left",(event.x)/2+"px")
           .style("top",(event.y)/2+"px")
  }
  const mouseleave = function(event, d) {
    tooltip
      .style("opacity", 0)
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


</script>

<main>
  <h1>Victor wembanyama`s performance in block and steal compare to the first round pick in 2023</h1>
  <h3>Nba data from october to feb 15</h3>
  <div id="chart"></div>
  <p>To compare Victor Wembenyama's performance with other players, our first step is to analyze his performance against first-round draft picks. We aim to implement a checkbox feature on the right side so users can select the data they want to compare for all the rookies. Currently, we have only completed the stacked bar plot for steals and blocks. Additionally, hovering over the bars will display the respective player's stats. While we're still working on the selection feature, we're also gathering the data we need to compare with Wembenyama to predict his future performances.
    
  <p>One of the most challenging aspects has been deciding which plots to use to present various NBA data. We intend to use a map to illustrate how Wembanyama performed against teams from different cities, allowing us to analyze his performance in different regions of the states. Furthermore, we plan to create a shooting map for Wembenyama to highlight his favorite offensive areas. However, collecting shooting data has proven difficult as there isn't an existing data frame we can use.</p>
    
  <p>Lastly, fetching the performance statistics of all first-round draft picks or the rookie stats of all Hall of Fame players has been challenging. This task requires combining multiple data frames.</p>
    
  
</main>

<style>
  /* Write your CSS here */
  h1 {text-align: left;}
  h3{
    text-align: middle;
    font-weight: normal;
  }
  
  
</style>
