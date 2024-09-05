<script>
  export let data;
  // export let xScale;
  // export let yScale;
  export let width;

  // $: x = xScale(data.grade);
  // $: y = yScale(data.hours);

  let tooltipWidth;

  let x = data.x;
  let y = data.y;

  const xNudge = 15;
  const yNudge = 30;

  $: xPosition =
    x + tooltipWidth > width ? x - tooltipWidth - xNudge : x + xNudge;
  $: yPosition = y + yNudge;

  import { fly } from "svelte/transition";
</script>

<div
  class="tooltip"
  transition:fly={{ y: 10 }}
  style="position: absolute; top: {yPosition}px; left: {xPosition}px"
  bind:clientWidth={tooltipWidth}
>
  <h1>{data.Title}</h1>
  <h2>{data.Year}</h2>
  <a href="https://www.example.com" class="image-link" target="_blank">
    <img src="{data.IMG}" alt="Poster">
</a>
</div>

<style>
  .tooltip {
    padding: 8px 6px;
    background: white;
    box-shadow: rgba(0, 0, 0, 0.15) 2px 3px 8px;
    border-radius: 3px;
    pointer-events: none;
    transition: top 300ms ease, left 300ms ease;
  }

  h1{
    margin: 0;
    padding: 0;
    font-weight: 300;
  }

  h2 {
    font-size: 1rem;
    font-weight: 400;
    margin-bottom: 6px;
    width: 100%;
  }
</style>
