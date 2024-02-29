<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from './Graph.svelte';

  let data = [];

  onMount(async () => {
    const csvPath = 'Indicator_3_1_Climate_Indicators_Annual_Mean_Global_Surface_Temperature_577579683071085080.csv';
    const res = await fetch(csvPath);
    const csv = await res.text();
    const rawData = d3.csvParse(csv);

    // Transform data from wide to long format
    data = rawData.reduce((acc, row) => {
      const { Country, ISO2, ISO3, Indicator, Unit, Source, ...years } = row;
      Object.keys(years).forEach(year => {
        acc.push({
          country: Country,
          year: +year,
          metric: +years[year]
        });
      });
      return acc;
    }, []);
    

    console.log(data); // To check the transformed data
  });
</script>

<main>
  <Graph {data} />
</main>