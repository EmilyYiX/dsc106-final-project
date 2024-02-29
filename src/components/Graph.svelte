<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data;

  let svg;

  onMount(() => {
    const margin = { top: 20, right: 20, bottom: 30, left: 50 };
    const width = 960 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;
    
    // Filter data for one country as an example
    const countryData = data.filter(d => d.country);

    const x = d3.scaleLinear()
                .domain(d3.extent(countryData, d => d.year))
                .range([0, width]);

    const y = d3.scaleLinear()
                .domain([d3.min(countryData, d => d.tempChange), d3.max(countryData, d => d.tempChange)])
                .range([height, 0]);

    const xAxis = d3.axisBottom(x).tickFormat(d3.format('d'));
    const yAxis = d3.axisLeft(y);

    const line = d3.line()
                   .x(d => x(d.year))
                   .y(d => y(d.tempChange));

    const svgElement = d3.select(svg)
                         .append('g')
                         .attr('transform', `translate(${margin.left},${margin.top})`);

    svgElement.append('g')
              .attr('transform', `translate(0,${height})`)
              .call(xAxis);

    svgElement.append('g')
              .call(yAxis);

    svgElement.append('path')
              .datum(countryData)
              .attr('fill', 'none')
              .attr('stroke', 'steelblue')
              .attr('stroke-width', 1.5)
              .attr('d', line);

    svgElement.append('text')
              .attr('text-anchor', 'end')
              .attr('x', width)
              .attr('y', height - 6)
              .style('font-size', '15px')
              .text('Year');

    svgElement.append('text')
              .attr('text-anchor', 'end')
              .attr('transform', 'rotate(-90)')
              .attr('y', 6)
              .attr('dy', '.75em')
              .style('font-size', '10px')
              .text('Temperature Change (°C)');

    svgElement.append('text')
              .attr('x', (width / 2))             
              .attr('y', 10 - (margin.top / 2))
              .attr('text-anchor', 'middle')  
              .style('font-size', '22px') 
              .text('Climate Change over Time Globally');
  });
</script>

<svg bind:this={svg} width="960" height="500"></svg>