<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import MultiSelect from 'svelte-multiselect';

  export let data;
  let svg;
  let displayedData = [];
  let countries = ["Afghanistan, Islamic Rep. of", "Albania", "Algeria", "American Samoa", "Andorra, Principality of", "Angola", "Anguilla", "Antigua and Barbuda", "Argentina", "Armenia, Rep. of", "Aruba, Kingdom of the Netherlands", "Australia", "Austria", "Azerbaijan, Rep. of", "Bahamas, The", "Bahrain, Kingdom of", "Bangladesh", "Barbados", "Belarus, Rep. of", "Belgium", "Belize", "Benin", "Bhutan", "Bolivia", "Bosnia and Herzegovina", "Botswana", "Brazil", "British Virgin Islands", "Brunei Darussalam", "Bulgaria", "Burkina Faso", "Burundi", "Cabo Verde", "Cambodia", "Cameroon", "Canada", "Cayman Islands", "Central African Rep.", "Chad", "Chile", "China, P.R.: Hong Kong", "China, P.R.: Macao", "China, P.R.: Mainland", "Colombia", "Comoros, Union of the", "Congo, Dem. Rep. of the", "Congo, Rep. of", "Cook Islands", "Costa Rica", "Croatia, Rep. of", "Cuba", "Cyprus", "Czech Rep.", "Denmark", "Djibouti", "Dominica", "Dominican Rep.", "Ecuador", "Egypt, Arab Rep. of", "El Salvador", "Equatorial Guinea, Rep. of", "Eritrea, The State of", "Estonia, Rep. of", "Eswatini, Kingdom of", "Ethiopia, The Federal Dem. Rep. of", "Falkland Islands (Malvinas)", "Faroe Islands", "Fiji, Rep. of", "Finland", "France", "French Polynesia", "Gabon", "Gambia, The", "Georgia", "Germany", "Ghana", "Gibraltar", "Greece", "Greenland", "Grenada", "Guadeloupe", "Guatemala", "Guinea", "Guinea-Bissau", "Guyana", "Haiti", "Holy See", "Honduras", "Hungary", "Iceland", "India", "Indonesia", "Iran, Islamic Rep. of", "Iraq", "Ireland", "Isle of Man", "Israel", "Italy", "Jamaica", "Japan", "Jordan", "Kazakhstan, Rep. of", "Kenya", "Kiribati", "Korea, Dem. People's Rep. of", "Korea, Rep. of", "Kuwait", "Kyrgyz Rep.", "Lao People's Dem. Rep.", "Latvia", "Lebanon", "Lesotho, Kingdom of", "Liberia", "Libya", "Liechtenstein", "Lithuania", "Luxembourg", "Madagascar, Rep. of", "Malawi", "Malaysia", "Maldives", "Mali", "Malta", "Marshall Islands, Rep. of the", "Martinique", "Mauritania, Islamic Rep. of", "Mauritius", "Mayotte", "Mexico", "Micronesia, Federated States of", "Moldova, Rep. of", "Monaco", "Mongolia", "Montenegro", "Montserrat", "Morocco", "Mozambique, Rep. of", "Myanmar", "Namibia", "Nauru, Rep. of", "Nepal", "Netherlands, The", "New Caledonia", "New Zealand", "Nicaragua", "Niger", "Nigeria", "Niue", "Norfolk Island", "North Macedonia, Republic of ", "Norway", "Oman", "Pakistan", "Palau, Rep. of", "Panama", "Papua New Guinea", "Paraguay", "Peru", "Philippines", "Pitcairn Islands", "Poland, Rep. of", "Portugal", "Puerto Rico", "Qatar", "Romania", "Russian Federation", "Rwanda", "Saint Helena", "Saint Pierre and Miquelon", "Samoa", "San Marino, Rep. of", "São Tomé and Príncipe, Dem. Rep. of", "Saudi Arabia", "Senegal", "Serbia, Rep. of", "Seychelles", "Sierra Leone", "Singapore", "Slovak Rep.", "Slovenia, Rep. of", "Solomon Islands", "Somalia", "South Africa", "South Sudan, Rep. of", "Spain", "Sri Lanka", "St. Kitts and Nevis", "St. Lucia", "St. Vincent and the Grenadines", "Sudan", "Suriname", "Sweden", "Switzerland", "Syrian Arab Rep.", "Taiwan Province of China", "Tajikistan, Rep. of", "Tanzania, United Rep. of", "Thailand", "Timor-Leste, Dem. Rep. of", "Togo", "Tokelau", "Tonga", "Trinidad and Tobago", "Tunisia", "Turkmenistan", "Turks and Caicos Islands", "Tuvalu", "Uganda", "Ukraine", "United Arab Emirates", "United Kingdom", "United States", "United States Virgin Islands", "Uruguay", "Uzbekistan, Rep. of", "Vanuatu", "Venezuela, Rep. Bolivariana de", "Vietnam", "Wallis and Futuna Islands", "West Bank and Gaza", "Western Sahara", "World", "Yemen, Rep. of", "Zambia", "Zimbabwe"]
  let selected = ['Brazil', 'Nigeria', 'Pakistan', 'Indonesia', 'United States', 'China, P.R.: Mainland', 'India'];
  let svgElement;
  let line;
  let x;
  let y;

  // Prepare the initial data filter
  let filteredData;

  function getFilteredData() {
    return data.filter(d => selected.includes(d.country));
  }

  // Function to restructure data for drawing lines
  function restructureData(data) {
    return d3.groups(data, d => d.country)
      .map(([country, values]) => ({ country, values }));
  }

  onMount(() => {
    filteredData = getFilteredData();
    const margin = { top: 20, right: 20, bottom: 30, left: 50 };
    const width = 1200 - margin.left - margin.right;
    const height = 600 - margin.top - margin.bottom;

    // Define scales
    x = d3.scaleLinear().domain(d3.extent(data, d => d.year)).range([0, width]);
    y = d3.scaleLinear().domain([d3.min(data, d => d.tempChange), d3.max(data, d => d.tempChange)]).range([height, 0]);

    // Define axes
    const xAxis = d3.axisBottom(x).tickFormat(d3.format('d'));
    const yAxis = d3.axisLeft(y);

    // Define the line generator
    line = d3.line()
      .x(d => x(d.year))
      .y(d => y(d.tempChange));

    // Append SVG container
    svgElement = d3.select(svg).append('g')
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

    d3.select(svg)
      .on("pointerenter", pointerentered)
      .on("pointerleave", pointerleft)
      .on("pointermove", pointermoved);
});

  function pointermoved(event) {
    if (selected.length == 0) {
      return;
    }
    const [xm, ym] = d3.pointer(event, this); // Get the mouse position relative to the SVG element
    let hoveredCountry = null;
    let closestDistance = Infinity;
    let closestYear = null;
    let closestTempChange = null;

    filteredData.forEach((e) => {
        const xCoord = x(e.year);
        const yCoord = y(e.tempChange);
        const distance = Math.sqrt((xm - xCoord) ** 2 + (ym - yCoord) ** 2);
        if (distance < closestDistance){
          closestDistance = distance;
          hoveredCountry = e.country;
          closestYear = e.year
          closestTempChange = e.tempChange;
        }
      }
    )

    if (hoveredCountry) {
      const xCoord = x(closestYear);
      const yCoord = y(closestTempChange);
      d3.select('#tooltip')
      .html(`Country: ${hoveredCountry}<br>Year: ${closestYear}<br>Temperature Change: ${closestTempChange}°C`)
      .style('left', `${xCoord + svg.getBoundingClientRect().x}px`)
      .style('top', `${yCoord + svg.getBoundingClientRect().y}px`)
      .style('opacity', 1);
      
    }
  }

  function pointerentered() {
    if (selected.length == 0) {
      return;
    }
    d3.select('#tooltip').style("opacity", 1);
  }

  function pointerleft() {
    d3.select('#tooltip').style("opacity", 0);
  }

  // Function to update the chart based on selected countries or search
  function updateChart(newData) {
    const lines = svgElement.selectAll('.line')
      .data(newData, d => d.country);

    lines.enter().append('path')
    .attr('class', 'line')
    .attr('d', d => line(d.values))
    .attr('fill', 'none')
    .attr('stroke', (d, i) => d3.schemeCategory10[i % 10]) // Optional: Use different colors for lines
    .attr('stroke-width', 1.5)

    // Update existing lines (if any)
    lines.attr('d', d => line(d.values));

    // Remove lines if their data was removed from displayedData (not required for your current functionality but useful if you implement data removal)
    lines.exit().remove();
  }

  function update(event){
    if (event.detail && event.detail.type == 'removeAll'){
      selected = [];
    }
    if (selected.length == 0) {
      d3.select('#tooltip').style("opacity", 0);
    }
    filteredData = getFilteredData();
    let restructuredData = restructureData(filteredData);
    updateChart(restructuredData);
  }

</script>
<div style="background-color: #5F5D9C">
  <h1 class="roboto-regular" style="display: block; margin: auto; text-align: center; color: #ffffff">How Has Climate Change Affected Temperature?</h1>
</div>
<div style="margin-top: 10px;"> <!-- Added margin-top to move the input box down -->
  <MultiSelect bind:selected options={countries} on:change={update} />
</div>
<div id="tooltip" class="roboto-regular" style="position: absolute; opacity: 0; background-color: rgba(255, 255, 255, 0.5); border: 1px solid #ddd; padding: 10px; border-radius: 4px; pointer-events: none; font-size: 14px;"></div>
<svg bind:this={svg} width="1200" height="600" class="roboto-regular" style="display: block; margin: auto"></svg>
<div class="roboto-regular">
  <h1>Writeup</h1>
  <h2>What have we done so far?</h2>
  <p>So far we have set up the project on GitHub and created a basic line plot. We have cleaned the dataset according to the 
    functionality that we want to implement. As part of cleaning we have dealt with missing values and column labels. We have also 
    implemented a search box to search for a country and see its temperature change over the years. That was one interactive element 
    we have implemented. Another interactive element that we have implemented is the tooltip information for the lines on the plot. It 
    shows the country and the temperature change of the nearest country's line.
  </p>
  <h2>What was the most challenging part to design?</h2>
  <p> The most intricate part of the project design was the creation of an efficient strategy for addressing missing values,
    particularly by devising a logic to backfill these voids with the latest available data on temperature change. This is pivotal
    for preserving the dataset's integrity and avoiding potential biases that could skew the analysis. Moreover, the implementation
    of an intuitive search feature that aligns smoothly with the data visualization is vital. It must accommodate user input
    without noticeable delays, ensuring a responsive and seamless experience. Finally, developing an innovative method to manage 
    a dynamic list of countries, which allows for easy editing, addition, and deletion, represents a significant UI/UX design
    challenge that necessitates a refined and strategic approach to both design and execution.
  </p>
</div>