<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import MultiSelect from 'svelte-multiselect';

  export let data;
  let svg;
  let displayedData = [];
  let countries = ['Afghanistan', 'Africa', 'Albania', 'Algeria', 'Andorra', 'Angola','Anguilla', 'Antigua and Barbuda', 'Argentina', 'Armenia', 'Aruba','Asia', 'Asia (excl. China and India)', 'Australia', 'Austria','Azerbaijan', 'Bahamas', 'Bahrain', 'Bangladesh', 'Barbados','Belarus', 'Belgium', 'Belize', 'Benin', 'Bermuda', 'Bhutan','Bolivia', 'Bonaire Sint Eustatius and Saba','Bosnia and Herzegovina', 'Botswana', 'Brazil','British Virgin Islands', 'Brunei', 'Bulgaria', 'Burkina Faso','Burundi', 'Cambodia', 'Cameroon', 'Canada', 'Cape Verde','Central African Republic', 'Chad', 'Chile', 'China', 'Colombia','Comoros', 'Congo', 'Cook Islands', 'Costa Rica', "Cote d'Ivoire",'Croatia', 'Cuba', 'Curacao', 'Cyprus', 'Czechia','Democratic Republic of Congo', 'Denmark', 'Djibouti', 'Dominica','Dominican Republic', 'East Timor', 'Ecuador', 'Egypt','El Salvador', 'Equatorial Guinea', 'Eritrea', 'Estonia','Eswatini', 'Ethiopia', 'Europe', 'Europe (excl. EU-27)','Europe (excl. EU-28)', 'European Union (27)','European Union (28)', 'Faroe Islands', 'Fiji', 'Finland','France', 'French Polynesia', 'Gabon', 'Gambia', 'Georgia','Germany', 'Ghana', 'Greece', 'Greenland', 'Grenada', 'Guatemala','Guinea', 'Guinea-Bissau', 'Guyana', 'Haiti','High-income countries', 'Honduras', 'Hong Kong', 'Hungary','Iceland', 'India', 'Indonesia', 'Iran', 'Iraq', 'Ireland','Israel', 'Italy', 'Jamaica', 'Japan', 'Jordan', 'Kazakhstan','Kenya', 'Kiribati', 'Kosovo', 'Kuwait', 'Kyrgyzstan', 'Laos','Latvia', 'Lebanon', 'Lesotho', 'Liberia', 'Libya','Liechtenstein', 'Lithuania', 'Low-income countries','Lower-middle-income countries', 'Luxembourg', 'Macao','Madagascar', 'Malawi', 'Malaysia', 'Maldives', 'Mali', 'Malta','Marshall Islands', 'Mauritania', 'Mauritius', 'Mexico','Micronesia (country)', 'Moldova', 'Mongolia', 'Montenegro','Montserrat', 'Morocco', 'Mozambique', 'Myanmar', 'Namibia','Nauru', 'Nepal', 'Netherlands', 'New Caledonia', 'New Zealand','Nicaragua', 'Niger', 'Nigeria', 'Niue', 'North America','North America (excl. USA)', 'North Korea', 'North Macedonia','Norway', 'Oceania', 'Oman', 'Pakistan', 'Palau', 'Palestine','Panama', 'Papua New Guinea', 'Paraguay', 'Peru', 'Philippines','Poland', 'Portugal', 'Qatar', 'Romania', 'Russia', 'Rwanda','Saint Helena', 'Saint Kitts and Nevis', 'Saint Lucia','Saint Pierre and Miquelon', 'Saint Vincent and the Grenadines','Samoa', 'Sao Tome and Principe', 'Saudi Arabia', 'Senegal','Serbia', 'Seychelles', 'Sierra Leone', 'Singapore','Sint Maarten (Dutch part)', 'Slovakia', 'Slovenia','Solomon Islands', 'Somalia', 'South Africa', 'South America','South Korea', 'South Sudan', 'Spain', 'Sri Lanka', 'Sudan','Suriname', 'Sweden', 'Switzerland', 'Syria', 'Taiwan','Tajikistan', 'Tanzania', 'Thailand', 'Togo', 'Tonga','Trinidad and Tobago', 'Tunisia', 'Turkey', 'Turkmenistan','Turks and Caicos Islands', 'Tuvalu', 'Uganda', 'Ukraine','United Arab Emirates', 'United Kingdom', 'United States','Upper-middle-income countries', 'Uruguay', 'Uzbekistan','Vanuatu', 'Venezuela', 'Vietnam', 'Wallis and Futuna', 'World','Yemen', 'Zambia', 'Zimbabwe']
  let selected = ['Brazil', 'Nigeria', 'Pakistan', 'Indonesia', 'United States', 'China', 'India'];
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
    y = d3.scaleLinear().domain([d3.min(data, d => d.co2Emissions), 30]).range([height, 0]);

    // Define axes
    const xAxis = d3.axisBottom(x).tickFormat(d3.format('d'));
    const yAxis = d3.axisLeft(y);

    // Define the line generator
    line = d3.line()
      .x(d => x(d.year))
      .y(d => y(d.co2Emissions));

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
              .text("CO2 Change");

    // Append chart title
    svgElement.append("text")
              .attr("x", (width / 2))             
              .attr("y", 10 - (margin.top / 2))
              .attr("text-anchor", "middle")  
              .style("font-size", "16px") 
              .style("text-decoration", "underline")  
              .text("CO2 Change Over Years Globally");

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
    let closestco2 = null;

    filteredData.forEach((e) => {
        const xCoord = x(e.year);
        const yCoord = y(e.co2Emissions);
        const distance = Math.sqrt((xm - xCoord) ** 2 + (ym - yCoord) ** 2);
        if (distance < closestDistance){
          closestDistance = distance;
          hoveredCountry = e.country;
          closestYear = e.year
          closestco2 = e.co2Emissions;
        }
      }
    )

    if (hoveredCountry) {
      const xCoord = x(closestYear);
      const yCoord = y(closestco2);
      d3.select('#tooltip')
      .html(`Country: ${hoveredCountry}<br>Year: ${closestYear}<br>CO2 Emissions: ${closestco2}°C`)
      .style('left', `${xCoord + svg.getBoundingClientRect().x}px`)
      .style('top', `${yCoord + 700}px`)
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
<div
style="
    background-color: #f2f2f2; /* Light grey background */
    border-left: 6px solid #2c3e50; /* Dark blue left border */
    font-family: 'Arial', sans-serif; /* Clean font choice */
    margin: 10px 0; /* Margin for spacing */
    padding: 10px; /* Padding for space inside the div */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Subtle shadow */
    border-radius: 4px; /* Rounded corners */
    line-height: 1.6; /* Improve line spacing */
"
>
  Our world is constantly changing and its not for the good. Extreme weathers like sheer cold or continuous burning days can be a reality. This is because the carbon dioxide emissions by us humans is causing temperature changes that is affecting the climate
</div>
</div>
<div style="margin-top: 10px;"> <!-- Added margin-top to move the input box down -->
  <MultiSelect bind:selected options={countries} on:change={update} />
</div>
<div id="tooltip" class="roboto-regular" style="position: absolute; opacity: 0; background-color: rgba(255, 255, 255, 0.5); border: 1px solid #ddd; padding: 10px; border-radius: 4px; pointer-events: none; font-size: 14px;"></div>
<svg bind:this={svg} width="1200" height="600" class="roboto-regular" style="display: block; margin: auto"></svg>
<div
style="
    background-color: #f2f2f2; /* Light grey background */
    border-left: 6px solid #2c3e50; /* Dark blue left border */
    font-family: 'Arial', sans-serif; /* Clean font choice */
    margin: 10px 0; /* Margin for spacing */
    padding: 10px; /* Padding for space inside the div */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Subtle shadow */
    border-radius: 4px; /* Rounded corners */
    line-height: 1.6; /* Improve line spacing */
"
>
  Looking at the trend of carbon dioxide emissions for the United states in the above visualization, we can observe that there are significant spikes in carbon dioxide emissions during the times 1960-70s, 1980-90's, and 2000-2006. Early 1960's was the period post world war II where Unites states saw increased manufacturing and industrial expansion. In 1962 we saw the "Silent Spring" in 1962 and Cuyahoga River fire in 1969. These could be reasons for the spikes during the 1960's.
  Early 1980's saw a gradual shift from coal to natural gas and nuclear energy, driven by environmental policies and economic factors. During this period United States saw that the carbon dioxide emissions were increasing and introduced amendments to the Clean Air Act in 1990 to address acid rain, toxic emissions, and ozone depletion. These significant events explain the spikes during this period.
  The early 2000s saw significant advances in technology, including the increased efficiency of vehicles and industrial processes. Vehicles and the energy released by technology plants are massive source of carbon dioxide emissions, explaining the spikes in the early 2000's
</div>