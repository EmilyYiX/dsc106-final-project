<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from './CO2Graph.svelte';

  let data = [];

  onMount(async () => {
    const response = await fetch('co-emissions-per-capita.csv');
    const text = await response.text();
    let rawData = d3.csvParse(text);

    // Transform data to a more manageable structure if needed
    let transformedData = transformData(rawData);

    // Fill missing data
    data = fillMissingValues(transformedData);
  });

 function transformData(rawData) {
  // Transform the data to include only country, year, and CO2 emissions
  return rawData.map(d => {
    return {
      country: d.Entity,
      year: +d.Year,
      co2Emissions: d['Annual CO₂ emissions (per capita)'] ? +d['Annual CO₂ emissions (per capita)'] : undefined // Convert to number, keep as undefined if missing
    };
  });
}

// This function fills missing CO2 emissions values
function fillMissingValues(data) {
  let finalData = [];
  // Group data by country
  let groupedByCountry = d3.group(data, d => d.country);

  groupedByCountry.forEach((values, country) => {
    let lastKnownValue = undefined;
    values.sort((a, b) => a.year - b.year); // Ensure the data is sorted by year
    for (let entry of values) {
      if (entry.co2Emissions !== undefined) {
        lastKnownValue = entry.co2Emissions;
        finalData.push(entry);
      } else if (lastKnownValue !== undefined) {
        entry.co2Emissions = lastKnownValue; // Fill missing value with last known value
        finalData.push(entry);
      }
    }
  });

  return finalData;
}
</script>

<main>
  {#if data.length > 0}
    <Graph {data} />
  {/if}
</main>
