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

    // Append X axis label
    svgElement.append("text")
              .attr("transform", `translate(${width / 2},${height + margin.top + 20})`)
              .style("text-anchor", "middle")
              .attr('transform', `translate(0,${height})`)
              .attr('x', width / 2)
              .attr('y', margin.bottom) 
              .text("Year");

    svgElement.append('g').call(yAxis);

    // Append Y axis label
    svgElement.append("text")
              .attr("transform", "rotate(-90)")
              .attr("y", 0 - margin.left)
              .attr("x", 0 - (height / 2))
              .attr("dy", "1em")
              .style("text-anchor", "middle")
              .text("Temperature Change (°C)");

    // Append chart title
    svgElement.append("text")
              .attr("x", (width / 2))             
              .attr("y", 10 - (margin.top / 2))
              .attr("text-anchor", "middle")  
              .style("font-size", "16px") 
              .style("text-decoration", "underline")  
              .text("Temperature Change (°C) Over Years Globally");

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
    .attr('stroke', (d, i) => d3.schemeCategory10[i % 10]) // Optional: Use different colors for lines
    .attr('stroke-width', 1.5)
    .on('mouseover', function() { d3.select('#tooltip').style('opacity', 1); })
    .on('mousemove', function(event, d) {
      const xPosition = d3.pointer(event, this)[0];
      const year = Math.round(x.invert(xPosition));
      const dataPoint = d.values.find(v => v.year === year);
      if (dataPoint) {
        d3.select('#tooltip')
      .html(`Country: ${d.country}<br>Year: ${dataPoint.year}<br>Temperature Change: ${dataPoint.tempChange}°C`)
      .style('left', `${event.pageX + 10}px`)
      .style('top', `${event.pageY + 10}px`)
      .style('opacity', 1);
      }
    })
    .on('mouseout', function() { d3.select('#tooltip').style('opacity', 0); });

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

<div style="margin-top: 20px;"> <!-- Added margin-top to move the input box down -->
  <input type="text" id="countrySearch" placeholder="Search countries..." bind:this={inputElement}>
</div>
<div id="tooltip" style="position: absolute; opacity: 0; background-color: #fff; border: 1px solid #ddd; padding: 10px; border-radius: 4px; pointer-events: none; font-size: 14px;"></div>
<svg bind:this={svg} width="960" height="500"></svg>
<h1>Writeup</h1>
<h2>What have we done so far?</h2>
<p>So far we have setup the project on github and created a basic line plot. We have cleaned the datasets according to the 
  functionality that we want to implement. As part of cleaning we have dealt with missing values and column labels. We have also 
  implemented a search box to search for a country and see its temperature change over the years. This is one interactive element 
  we have implemented. Another interactive element that we have implemented is the tooltip information for the lines on plot. It 
  shows the country and the temperature change when the cursor is hovered over the line.
</p>
<h2>The most challenging part to design</h2>
<p> The most challenging aspect of the project to design is likely to be the development of an effective mechanism for handling
 missing values, specifically the logic of backfilling these gaps with the most recent year's temperature change data. 
 This process is not only crucial for maintaining the integrity of the dataset but also poses a risk of introducing biases
  that could distort the analysis. Additionally, the integration of a search box that interacts seamlessly with the visualization
   requires careful consideration of both the user interface and the underlying data handling, ensuring that the system can respond
    dynamically to user queries without performance lags. Lastly, crafting a new technique to dynamically display an editable 
    list of countries, complete with a functionality to add or remove entries, presents a significant UI/UX challenge that 
    demands a sophisticated approach to design and implementation.
</p>