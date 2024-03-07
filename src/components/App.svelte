<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from './Graph.svelte'; // Adjust according to your file structure

  let data = [];

  onMount(async () => {
    const response = await fetch('Indicator_3_1_Climate_Indicators_Annual_Mean_Global_Surface_Temperature_577579683071085080.csv');
    const text = await response.text();
    let rawData = d3.csvParse(text);

    // Transform data to a more manageable structure if needed
    let transformedData = transformData(rawData);

    // Fill missing data
    data = fillMissingValues(transformedData);
  });

  // Assuming rawData is an array of objects where each object is a row in the CSV
  function transformData(rawData) {
    // Transform the data as per your requirements
    // The following is a placeholder transformation
    return rawData.map(d => {
      const result = [];
      Object.keys(d).forEach(key => {
        if (!isNaN(Date.parse(key))) { // Ensure the key is a year
          result.push({
            country: d.Country,
            year: +key,
            tempChange: d[key] ? +d[key] : undefined // Convert to number, keep as undefined if missing
          });
        }
      });
      return result;
    }).flat(); // Flatten the array of arrays
  }

  function fillMissingValues(data) {
    let finalData = [];
    // Group data by country
    let groupedByCountry = d3.group(data, d => d.country);

    groupedByCountry.forEach((values, country) => {
      let lastKnownValue = undefined;
      values.sort((a, b) => a.year - b.year); // Ensure the data is sorted by year
      for (let entry of values) {
        if (entry.tempChange !== undefined) {
          lastKnownValue = entry.tempChange;
          finalData.push(entry);
        } else if (lastKnownValue !== undefined) {
          entry.tempChange = lastKnownValue; // Fill missing value with last known value
          finalData.push(entry);
        }
      }
    });

    // Convert the Map back to array format
    return finalData;
  }
</script>

<main>
  {#if data.length > 0}
    <Graph {data} />
  {/if}
</main>

