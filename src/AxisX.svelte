<script>
    // Importing the fade transition from Svelte
    import { fade } from "svelte/transition";

    // Exporting xScale and height to make them accessible outside the component
    export let xScale;
    export let height;
    export let scaleBand;
    
    // Generate xTicks using the ticks() method if the scale is not of type `scaleBand`
    let xTicks;
    if (scaleBand) {
        xTicks = xScale.domain();
    } else {
        xTicks = xScale.ticks(10);
    }
</script>

<!-- Group element for x-axis ticks -->
<g class="xTicks" transform="translate(0, {height})">
    {#each xTicks as tick, index}
        <g transform="translate({xScale(tick)}, 0)">
            <text x="0" y="0" dy="20" dominant-baseline="middle" text-anchor="middle" transition:fade={{ delay: index * 1000 }}>
                {tick}
            </text>
            <line x1="0" y1="0" x2="0" y2="10" stroke="rgba(0, 0, 0, 0.25)" />
        </g>
    {/each}
</g>

<style>
    .xTicks text {
        font-size: 10px;
        fill: rgba(0, 0, 0, 0.25);
    }

    .xTicks line {
        fill: rgba(0, 0, 0, 0.25);
    }
</style>
