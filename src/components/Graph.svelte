<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data;
  let svg;
  let displayedData = [];
  let inputElement; // Reference for the input element
  let initialCountries = ['Brazil', 'Nigeria', 'Pakistan', 'Indonesia', 'United States', 'China', 'India'];

  // Prepare the initial data filter
  let filteredData = data.filter(d => initialCountries.includes(d.country));

  // Function to restructure data for drawing lines
  function restructureData(data) {
    return d3.groups(data, d => d.country)
      .map(([country, values]) => ({ country, values }));
  }

  onMount(() => {
    const margin = { top: 20, right: 20, bottom: 30, left: 50 };
    const width = 960 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;

    // Define scales
    const x = d3.scaleLinear().domain(d3.extent(data, d => d.year)).range([0, width]);
    const y = d3.scaleLinear().domain([d3.min(data, d => d.tempChange), d3.max(data, d => d.tempChange)]).range([height, 0]);

    // Define axes
    const xAxis = d3.axisBottom(x).tickFormat(d3.format('d'));
    const yAxis = d3.axisLeft(y);

    // Define the line generator
    const line = d3.line()
                   .x(d => x(d.year))
                   .y(d => y(d.tempChange));

    // Append SVG container
    const svgElement = d3.select(svg).append('g')
                         .attr('transform', `translate(${margin.left},${margin.top})`);

    // Append axes to the SVG container
    svgElement.append('g')
              .attr('transform', `translate(0,${height})`)
              .call(xAxis);
    svgElement.append('g').call(yAxis);

    // Initial chart update with filtered data
    updateChart(restructureData(filteredData));

    // Listen to country selector changes (if you have a selector in your HTML)
    d3.select('#countrySelector').on('change', onCountryChange);

    // Function to update the chart based on selected countries or search
    function updateChart(newData) {
  // Merge newData with displayedData
  newData.forEach(newDataItem => {
    if (!displayedData.some(item => item.country === newDataItem.country)) {
      displayedData.push(newDataItem);
    }
  });

  // Now, use displayedData to update the chart
  const lines = svgElement.selectAll('.line')
    .data(displayedData, d => d.country);

  lines.enter().append('path')
    .attr('class', 'line')
    .attr('d', d => line(d.values))
    .attr('fill', 'none')
    .attr('stroke', 'steelblue')
    .attr('stroke-width', 1.5);

  // Update existing lines (if any)
  lines.attr('d', d => line(d.values));

  // Remove lines if their data was removed from displayedData (not required for your current functionality but useful if you implement data removal)
  lines.exit().remove();
}

    // Event handler for country selection changes
    function onCountryChange() {
      const selectedCountries = d3.select(this).selectAll('option:checked').data();
      const updatedData = data.filter(d => selectedCountries.includes(d.country));
      updateChart(restructureData(updatedData));
    }

    // Event handler for filtering countries via search
    function filterDataAndDraw(event) {
      const searchTerm = event.target.value;
      const updatedData = data.filter(d => d.country.toLowerCase().includes(searchTerm.toLowerCase()));
      // Assuming updateChart and restructureData are also defined or accessible here
      updateChart(restructureData(updatedData));
    }

    // Add the event listener to the input element
    function handleKeyPress(event) {
  if (event.keyCode === 13) {
    const searchTerm = event.target.value.trim();
    if (searchTerm) {
      const updatedData = data.filter(d => d.country.toLowerCase() === searchTerm.toLowerCase());
      const structuredData = restructureData(updatedData); // Ensure this matches the expected structure
      updateChart(structuredData); // This will now merge the new data
    }
  }
}

    // Change to listen for 'keyup' events
    inputElement.addEventListener('keyup', handleKeyPress);

    // Cleanup
    return () => {
      inputElement.removeEventListener('keyup', handleKeyPress);
    };
  });
</script>

<input type="text" id="countrySearch" placeholder="Search countries..." bind:this={inputElement}>
<svg bind:this={svg} width="960" height="500"></svg>
