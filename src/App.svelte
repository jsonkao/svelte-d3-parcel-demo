<script>

  import BubbleChart from './BubbleChart.svelte';
  import LineChart from './LineChart.svelte';

  import stocksRaw from '../data/stocks.json';
  // import flare from '../data/flare.json';

  const stocks = stocksRaw
    .map(d => ({ date: new Date(d.date), close: d.close }))
    .sort((a, b) => a.date - b.date);

</script>

<BubbleChart data={stocks}/>

<LineChart allData={stocks}/>
