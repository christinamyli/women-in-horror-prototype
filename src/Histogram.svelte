<script>
	import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
		import { scaleSqrt } from "d3-scale";
		import AxisX from './AxisX.svelte'

	//export statements
    // Exporting variables to make them accessible outside the component
    export let width; 
    export let height; 
    export let margin; 
    export let simulation; // The force simulation object to update circle positions
    export let X; // Function to determine the x-position of each circle
    export let Y; // Function to determine the y-position of each circle
    export let alpha; // Alpha value used for the force simulation
    export let adecay; // Alpha decay value used for the force simulation
    export let circleRadius; // Radius of the circles in the scatter plot
    export let xScale; // D3 scale function for mapping data to the x-axis
    export let chartHeight; // Height of the chart area within the SVG container
	
    // Declarations
    let nodes = []; // An array to store the data of circles (nodes) in the scatter plot

	simulation 	=  simulation
	    .force("x", forceX().x(X).strength(0.1))
	    .force("y", forceY().y(Y).strength(0.2))
	
  // Force simulation setup 

    // Setting up the force simulation for the scatter plot
    simulation = simulation
        .force("x", forceX().x(X).strength(0.1))
        .force("y", forceY().y(Y).strength(0.2));
	    // Apply forces to the x and y positions of the circles using the specified functions X and Y
	    // The strength parameter controls how much the forces affect the circles' movements

    // Force simulation setup
    simulation
        .force("collide", forceCollide().radius(circleRadius))
        .alpha(alpha)
        .alphaDecay(adecay)
        .restart();
			// Add a collision force to prevent circles from overlapping using the specified circleRadius
	    // The alpha and alphaDecay parameters control the simulation's internal time steps

    // Function to update circle positions on each tick of the simulation
    simulation.on("tick", () => {
        nodes = simulation.nodes(); // Update the nodes array with the latest circle positions
    });
</script>

<!-- SVG container for the scatter plot -->
<div class='chart-container'>
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
			<!-- Include the AxisX component to draw the x-axis with ticks -->
      <AxisX scaleBand={true} {xScale} height={chartHeight} />

      {#each nodes as node}
        <circle cx={node.x} cy={node.y} r={circleRadius} />
      {/each}
    </g>
  </svg>
</div>


<style>
circle{
		stroke:white
	}
 .inner-chart {
    top: 50%;
    left: 50%;
    transform: translate(15%, 0%);  
  }
</style>
