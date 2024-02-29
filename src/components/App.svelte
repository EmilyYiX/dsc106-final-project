<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from './Graph.svelte'; // Ensure this path matches your file structure

  let data = [];

  onMount(async () => {
    const response = await fetch('Indicator_3_1_Climate_Indicators_Annual_Mean_Global_Surface_Temperature_577579683071085080.csv');
    const text = await response.text();
    const parsed = d3.csvParse(text, d => {
      const result = [];
      Object.keys(d).forEach(key => {
        if (!isNaN(Date.parse(key))) { // Ensure the key is a year
          result.push({
            country: d.Country,
            year: +key,
            tempChange: +d[key]
          });
        }
      });
      return result;
    });
    data = parsed.flat(); // Flatten the array of arrays
  });
</script>

<main>
  {#if data.length > 0}
    <Graph {data} />
  {/if}
</main>

