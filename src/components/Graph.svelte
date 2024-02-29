<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  export let data;

  let svg;
  const margin = { top: 20, right: 20, bottom: 30, left: 50 },
        width = 960 - margin.left - margin.right,
        height = 500 - margin.top - margin.bottom;
  
  onMount(() => {
    if (data && data.length > 0) {
      const svgElement = d3.select(svg)
                           .attr('viewBox', `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
                           .append('g')
                           .attr('transform', `translate(${margin.left},${margin.top})`);

      const xScale = d3.scaleLinear()
                       .domain(d3.extent(data, d => d.year))
                       .range([0, width]);

      const yScale = d3.scaleLinear()
                       .domain([-3,5])
                       .nice()
                       .range([height, 0]);

      const xAxis = d3.axisBottom(xScale).tickFormat(d3.format('d'));
      const yAxis = d3.axisLeft(yScale);

      svgElement.append('g')
                 .attr('transform', `translate(0,${height})`)
                 .call(xAxis);

      svgElement.append('g')
                 .call(yAxis);

      // Group data by country
      const dataGrouped = d3.group(data, d => d.country);

      const lineGenerator = d3.line()
                              .x(d => xScale(d.year))
                              .y(d => yScale(d.metric));

      // Color scale for different countries
      const color = d3.scaleOrdinal(d3.schemeCategory10);

      // Draw lines for each country
      svgElement.selectAll('.line')
                 .data(dataGrouped)
                 .enter()
                 .append('path')
                 .attr('fill', 'none')
                 .attr('stroke', (d, i) => color(i))
                 .attr('stroke-width', 2)
                 .attr('d', d => lineGenerator(d[1]));
    }
  });
</script>

<svg bind:this={svg} width="960" height="500"></svg>