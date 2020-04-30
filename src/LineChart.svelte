<script>

  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

  import { select } from 'd3-selection';
  import { line } from 'd3-shape';
  import { scaleTime, scaleLinear } from 'd3-scale';
  import { extent } from 'd3-array';
  import { axisBottom, axisLeft } from 'd3-axis';
  import 'd3-transition';

  const width = window.innerWidth / 3;
  const height = 400;
  const margin = { top: 50, bottom: 50, left: 50, right: 50 };

  let xScale = scaleTime().range([ 0, width - margin.left - margin.right ]);
  let yScale = scaleLinear().range([ height - margin.bottom - margin.top, 0 ]);
  let lineFn = line();
  let xAxisFn = axisBottom().ticks(width / 100);
  let yAxisFn = axisLeft();

  let xAxis, yAxis;

  export let allData;

  let n = allData.length;
  let data;

  $: {

    data = allData.filter((_, i) => i < n);

    xScale = xScale.domain(extent(data, d => d.date));
    yScale = yScale.domain(extent(data, d => d.close));
    lineFn = lineFn.x(d => xScale(d.date)).y(d => yScale(d.close));

    xAxisFn = xAxisFn.scale(xScale);
    yAxisFn = yAxisFn.scale(yScale);

    xAxis && select(xAxis).transition().call(xAxisFn);
    yAxis && select(yAxis).transition().call(yAxisFn);

  }

  let path, pathLength;
  $: path && (pathLength = path.getTotalLength());

  let isVisible = true;

  let canvas;
  $: context = canvas && canvas.getContext('2d');
  $: {
    if (context) {
      context.clearRect(0, 0, canvas.width, canvas.height);
      context.strokeStyle = 'blue'
      lineFn.context(context);
      context.beginPath();
      lineFn(data);
      context.stroke();
    }
  }

</script>

<h2>Line Chart</h2>

<div class="container">

  <div>
    <p>Canvas</p>
    <canvas bind:this={canvas} width={width} height={height}></canvas>
  </div>

  <div>
    <p>SVG</p>
    <svg viewBox="0 0 {width} {height}" width={width} height={height}>
      <g transform="translate({margin.left}, {margin.top})">
        {#if isVisible}
          <path
            bind:this={path}
            transition:fade={{ duration: 200 }}
            class=line
            d={lineFn.context(null)(data)}
            fill=none
            stroke=steelblue
            stroke-width=1.5
          ></path>
        {/if}
        <g bind:this={xAxis} transform="translate(0, {height - margin.bottom - margin.top})"></g>
        <g bind:this={yAxis}></g>
      </g>
    </svg>
  </div>

</div>

<div>
  <p>
    <input type=range bind:value={n} min=1 max={allData.length}/>
    n = {n}
  </p>
  <p>
    <input type=checkbox bind:checked={isVisible}/>
    is SVG line visible?
  </p>
</div>

<style>
  .container {
    display: flex;
  }

  .line {
    vector-effect: non-scaling-stroke;
  }
</style>
