<script>
	import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
		import { scaleSqrt } from "d3-scale";
	  import { onMount } from "svelte";

		import AxisX from './AxisX.svelte'
		import AxisY from './AxisY.svelte'

	//export statements
		export let width;
		export let height;
		export let margin;
		export let simulation;
		export let X;
		export let Y;
	
		export let xScale;
		export let yScale;
		export let chartHeight;
		export let chartWidth;


	//Declarations
		let nodes = [];
		const radiusScale = scaleSqrt().domain([0, 100]).range([0, 30]);

	simulation 	=  simulation
	    .force("x", forceX().x(X).strength(0.1))
	    .force("y", forceY().y(Y).strength(0.2))
			.force("collide", forceCollide().radius(0))
			.alpha(0.2) 
	    .alphaDecay(0)
			.restart()


		// FUNCTIONS
  simulation.on("tick", () => {
    nodes = simulation.nodes(); // Update the nodes array
  });

	  // Trigger fade-in effect for the axis components after 1 second delay
  let showAxes = false;
  onMount(() => {
    setTimeout(() => {
      showAxes = true;
    }, 1000);
  });

	// Custom fade transition
  const fade = (node, { delay = 400, duration = 200 }) => {
    return {
      delay,
      duration,
      css: (t) => `opacity: ${t}`,
    };
  };
</script>

<div class="chart-container">
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      {#if showAxes}
        <AxisX scaleBand={false} {xScale} height={chartHeight} />
        <AxisY {yScale} width={chartWidth} />
      {/if}
      
      {#each nodes as node}
        <circle cx={node.x} cy={node.y} r={10} />
      {/each}
    </g>
  </svg>
</div>

<style>
circle{
		stroke:white
	}
</style>
