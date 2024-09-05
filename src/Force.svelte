<script>
  // Import necessary libraries and components
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleSqrt, scaleOrdinal } from "d3-scale";
  import Tooltip from "./Tooltip.svelte";
  import { onMount, afterUpdate } from 'svelte';

  // Exporting variables to make them accessible outside the component
  export let width; 
  export let height; 
  export let margin; 
  export let simulation; // D3 force simulation instance
  export let X; // Function to determine x-position of each circle in the simulation
  export let Y; // Function to determine y-position of each circle in the simulation
  export let alpha; 
  export let adecay; 
  export let circleRadius; // Either an integer or a function to determine the radius of each circle

  export let nodes = [];

  export let fills;

  let hoveredData;

  

  // Declarations
  // let nodes = []; // Array to store data for each circle (node) in the simulation
  const radiusScale = scaleSqrt().domain([0, 100]).range([0, 30]); // Scale for circle radius
  // const colorRange = [
  // "#dda0dd",
  // "#fe7f2d",
  // "#fcca46",
  // "#a1c181",
  // "#619b8a",
  // "#eae2b7",
  // ];

  const colorRange = [
  "#C6458D",
  "#808080",
  "#044892",
  "#D0842E",
  "#00502F"
  ];

  const colorScale =
  fills == "Role_Code"
    ? scaleOrdinal().domain("1", "2", "3", "4", "5", "6").range(colorRange)
    : scaleOrdinal().domain("1", "2").range(colorRange);

  // Set up force simulation with x and y forces
  simulation = simulation
      .force("x", forceX().x(X).strength(0.2)) // X-axis force based on the X function and strength
      .force("y", forceY().y(Y).strength(.2)) // Y-axis force based on the Y function and strength
      .force("collide", forceCollide().radius(circleRadius) + 2)

  // Force simulation setup
  if (Number.isInteger(circleRadius)) {
      // If circleRadius is an integer, apply the specified collision force
      simulation
          .force("collide", forceCollide().radius(circleRadius)) // Collision force with a fixed radius
          .alpha(alpha) // Set the initial alpha value for the simulation
          .alphaDecay(adecay) // Set the alpha decay rate for the simulation
          .restart(); // Restart the simulation with the updated forces
  } else {
      // If circleRadius is a function, apply a collision force based on the radiusScale
      simulation
          .force("collide", forceCollide().radius(d => radiusScale(d[circleRadius]))) // Collision force with a variable radius
          .alpha(alpha) // Set the initial alpha value for the simulation
          .alphaDecay(adecay) // Set the alpha decay rate for the simulation
          .restart(); // Restart the simulation with the updated forces
  }

  // Function to update circle positions on each tick of the simulation
  simulation.on("tick", () => {
      nodes = simulation.nodes(); // Update the nodes array with the latest circle positions
  });
</script>

<!-- SVG container for the scatter plot -->
<div class='chart-container'>
  <svg {width} {height} on:mouseleave={() => hoveredData = null}>
      <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
          <!-- Use the each block to iterate over the nodes array and draw circles for each data point -->
          {#each nodes as node}
              console.log(node)
              <!-- Set the cx (x-coordinate) and cy (y-coordinate) attributes based on the node's position in the simulation -->
              <!-- The r attribute is set either to the fixed circleRadius value or calculated using the radiusScale -->
              <circle cx={node.x} cy={node.y} r={Number.isInteger(circleRadius) ? circleRadius : radiusScale(node[circleRadius])}
              opacity={hoveredData ? (hoveredData == node ? 1 : 0.45) : 0.85}
              fill={fills == "#808080" ? fills : colorScale(node[fills])}
              on:mouseover={() => hoveredData = node}
              on:focus={() => hoveredData = node}/>
          {/each}
      </g>
  </svg>
  {#if hoveredData}
  <Tooltip {width} data={hoveredData} />
{/if}
</div>
<!-- 
fill={colorScale(node)} -->

<style>
  /* Add any custom styling for the chart here, if needed */
circle{
  stroke:white
}

@media screen and (max-width: 768px) {
  .chart-container {
    width: 90%; /* Slightly reduced width on mobile */
    overflow: hidden;
  }

  .chart-container svg {
    transform: scale(0.9); /* Slightly reduce size */
    transform-origin: center;
  }

  g.inner-chart {
    transform: scale(0.8); /* Adjust this value as needed to center the chart */
    transform-origin: center;
  }
}

</style>