<script>
    // Exporting yScale and width to make them accessible outside the component
    export let yScale;
    export let width;

    // Generate yTicks using the ticks() method
    let yTicks = yScale.ticks(4);
</script>

<!-- Group element for y-axis ticks -->
<g>
    {#each yTicks as tick}
        <g transform="translate(0, {yScale(tick)})">
            <line x1={0} y1={0} x2={width} y2={0} stroke="#c4c4c4" stroke-width="0.5" />
            <text x={0} y={-6}>{tick}</text>
        </g>
    {/each}
</g>

<style>
    /* Styling for the text elements */
    text {
        font-size: 10px;
        fill: rgba(0, 0, 0, 0.25);
    }
</style>
