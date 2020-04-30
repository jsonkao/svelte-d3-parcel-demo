<script>

  import { onMount } from 'svelte';
  import { forceSimulation, forceX, forceY, forceCollide } from 'd3-force';
  import { scaleSqrt } from 'd3-scale';
  import { extent } from 'd3-array';

  export let data;

  const width = document.body.clientWidth - 30;
  const height = 200;

  const centerX = width / 2;
  const centerY = height / 2;

  // Scale for the radius of a data point
  const radiusScale = scaleSqrt()
    .domain(extent(data, d => d.close))
    .range([ 2, 20 ]);

  // x forces for the simulation
  const forceXCombine = forceX(centerX).strength(0.05);
  const forceXSeparate = forceX(d => {
    if (d.date < new Date('2012-04-15'))
      return centerX - width / 4;
    return centerX + width / 4;
  }).strength(0.05);

  // Apply forces directly to the data. To trigger a re-render at every tick,
  // we must re-assign data.
  let simulation = forceSimulation(data)
    // Bring circles to the middle
    .force('x', forceXCombine)
    .force('y', forceY(centerY).strength(0.05))
    // Setup a collision force, which prevents overlap
    .force('collide', forceCollide(d => radiusScale(d.close) + 1).strength(0.9))
    .on('tick', () => data = data);

  let isSplit = false;
  let isFirstRender = true; // Prevents rehead nodes on initial rendering
  $: {
    if (isFirstRender) {
      isFirstRender = false;
    } else {
      // The nodes have lost energy, so when we change the simulation we must
      // "reheat" the nodes with alphaTarget and restart
      simulation = simulation
        .force('x', isSplit ? forceXSeparate : forceXCombine)
        .alphaTarget(0.6)
        .restart();
    }
  }

  let canvas;
  $: context = canvas && canvas.getContext('2d');
  $: {
    if (context) {
      context.clearRect(0, 0, canvas.width, canvas.height);
      context.fillStyle = 'red';
      data.forEach(d => {
        context.beginPath();
        context.arc(d.x, d.y, radiusScale(d.close), 0, 2 * Math.PI);
        context.fill();
      });
    }
  }

</script>

<h2>Bubble chart</h2>

<p>Canvas</p>
<canvas bind:this={canvas} width={width} height={height}></canvas>

<p>SVG</p>
<div id="chart">
  <svg width={width} height={height}>
    <!-- This transformation always centers the nodes in the SVG -->
    <g transform="translate({width / 2 - centerX}, {height / 2 - centerY})">
      {#each data as d (+d.date)}
        <circle
          r={radiusScale(d.close)}
          cx={d.x}
          cy={d.y}
          fill=red
        />
      {/each}
    </g>
  </svg>
</div>

<button on:click={() => isSplit = !isSplit}>
  {isSplit ? 'Combine groups' : 'Split groups'}
</button>
