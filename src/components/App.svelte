<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import * as d3 from 'd3';
  import Consumption from './Consumption.svelte';
  import EnergySpecific from './EnergySpecific.svelte';

  let data = [];
  let inputText = '';
  let uniqueCountries = [];
  let selectedCountries = [];
  let dropdownOpen = false;
  let tooltipPt = {};
  
  const continentList = ['North America', 'South America', 'Europe', 'Africa', 'Asia', 'Oceania', 'World'];

  onMount(async () => {
    const res = await fetch('owid-energy-data.csv');
    const csv = await res.text();
    data = await d3.csvParse(csv, d => ({
      country: d['country'],
      year: +d['year'],
      iso_code: d['iso_code'],
      population: +d['population'],
      gdp: +d['gdp'],
      biofuel_consumption: Math.round(+d['biofuel_cons_per_capita']),
      coal_consumption: Math.round(+d['coal_cons_per_capita']),
      fossil_fuel_consumption: Math.round(+d['fossil_energy_per_capita']),
      gas_consumption: Math.round(+d['gas_energy_per_capita']),
      hydro_consumption: Math.round(+d['hydro_consumption']),
      low_carbon_consumption: Math.round(+d['low_carbon_energy_per_capita']),
      nuclear_consumption: Math.round(+d['nuclear_energy_per_capita']),
      oil_consumption: Math.round(+d['oil_energy_per_capita']),
      other_renewable_consumption: Math.round(+d['other_renewables_energy_per_capita']),
      renewables_consumption: Math.round(+d['renewables_energy_per_capita']),
      solar_consumption: Math.round(+d['solar_energy_per_capita']),
      wind_consumption: Math.round(+d['wind_energy_per_capita']),
      prim_cons_per_capita: +d['energy_per_capita'],
    }));
    data = data.filter(d => d.iso_code !== '' || continentList.includes(d.country));
    uniqueCountries = [...new Set(data.map(d => d.country))];
  });

  function handleInput(event) {
    inputText = event.target.value.toLowerCase();
  }

  function handleCountrySelect(country) {
    if (!selectedCountries.includes(country) && (selectedCountries.length < 5)) {
      selectedCountries = [...selectedCountries, country];
      inputText = '';
      dropdownOpen = false;
    } else {
      alert("Not able to view more than 5 countries at once. Please remove a country before proceeding.")
    }
  }
  
  function removeSelectedCountry(country) {
    selectedCountries = selectedCountries.filter(selectedCountry => selectedCountry !== country);
  }

  function toggleDropdown() {
    dropdownOpen = !dropdownOpen;
  }

  function closeDropdown(event) {
    if (dropdownOpen && !event.target.closest('.dropdown')) {
      dropdownOpen = false;
    }
  }

  function getColorClass(country) {
      const index = selectedCountries.indexOf(country);
      const colors = ["#648FFF", "#FE6100", "#DC267F", "#FFB000" ,"#785EF0"];
      return colors[index % colors.length];
  }

  function handleDataUpdate(event) {
    tooltipPt = event.detail;
  }
</script>

<main on:click={closeDropdown}>
  <h1 style="line-height: 1.25">Do Trends in Energy Consumption Per Capita Differ Across Countries, and<br>What Are the Leading Energy Sources?</h1>
  <div style="margin-top: 10px;"  class="dropdown" on:click={toggleDropdown}>
    <input type="text" placeholder="Search a country" on:input={handleInput} bind:value={inputText}>
    <div class="dropdown-content" style="display: {dropdownOpen ? 'block' : 'none'}">
      {#if inputText !== ''}
        {#each uniqueCountries.filter(country => country.toLowerCase().includes(inputText) && !selectedCountries.includes(country)) as country}
          <div class="dropdown-item" on:click={() => handleCountrySelect(country)}>{country}</div>
        {/each}
      {/if}
    </div>
  </div>

  <div class="selected-countries">
    {#each selectedCountries as country}
      <div class="selected-country" style="color: {getColorClass(country)}">
        <input type="checkbox" class="done" on:click={() => removeSelectedCountry(country)}>{country}<br>
      </div>
    {/each}
  </div>

  <div class="container">
    <Consumption on:dataUpdate={handleDataUpdate} {data} {selectedCountries} />
    <EnergySpecific {tooltipPt} {selectedCountries} />
  </div>

</main>
<br><br><br><hr><br><br>
<div class="writeup">
  <h2>Project3 Write-up</h2>
  <p>
    For our project, we chose to use the Data on Energy by Our World in Data, 1900-2022, from Our World In Data. The goal of our visualization was to see how primary energy consumption per capita breaks down over the years compared among countries. Our main visualization is a line plot due to its effectiveness in showing trends over the years. Since we wanted to see how the trends differ across countries, we added a search function for the user to view and compare up to 5 countries at a time. The line plot shows the given countries' primary energy consumption per capita (kWh per person) on the y-axis and years on the x-axis which are each from the `energy_per_capita` and `year` columns. We also added a dashed line of world energy consumption that is constantly viewed so that the user can get a sense of how each country’s trend is different from the world average. For this line, we didn’t have to calculate it since it was already in the dataset with the country name column as World. Once the viewer adds a country through the search bar at the top of the line plot, the given country is added to the visualization, and a container is added above to keep track and act as a legend. We color-coded the title of the country to the line on the plot for clarity with the colorblind color palette established by IBM. Also, we added a button next to the country for easy removal and editing of the countries visualized in the plot. If the user attempts to add more than 5 countries an alert popups explaining to the user that a country needs to be removed before adding a new one. We set this limit since it would be difficult to compare lots of lines with different colors at the same time. 
  </p><br>
  <p>
    Before we arrived at this final line plot, we considered adding all the countries on the plot before the user searched for a country. However, because of the number of unique countries in the dataset, it took too long to show something initially to the user, so we disregarded the idea. We also thought about adding a tooltip interaction to highlight one line that’s being hovered over. In fact, this was the implementation that took the longest since we ran into several issues with intersecting lines and the tooltip obstructing other data on the line plot. So we tried making a tooltip to have a semi-transparent background color, positioning it at a fixed point inside the line plot, and reducing the size, but we thought none of them were good at showing both data effectively. Thus instead of having a tooltip directly on the visualization, we opted for a second interactive visualization: a horizontal bar plot that breaks down energy consumption by type for the given year and country hovered over on the line plot. The bar plot has energy consumption per capita categories on the y-axis and percentages on the x-axis. These values are from `biofuel_cons_per_capita`, `coal_cons_per_capita`, `fossil_energy_per_capita`, `gas_energy_per_capita`, `hydro_consumption`, `low_carbon_energy_per_capita`, `nuclear_energy_per_capita`, `oil_energy_per_capita`, `solar_energy_per_capita`, and `wind_energy_per_capita` columns which are in a form of kWH/per person. Thus, we needed to calculate the percentage of each category to determine the width and percentage values of the bars. To visually connect the specific line plot and the bar plot when the user interacts with our visualization, we used the same color. Then, we added a circle to the line plot with the same color as the line plot being hovered over to indicate which year and country the user is hovering over so that it gives feedback to the user on the interactivity. We thought about also adding a horizontal and vertical gray dashed line along with the circle, but we eventually didn’t since we thought it would be visually distracting considering the amount of data we already presented to the user. Also, to emphasize the line plot more than the bar plot, we allocate 70% of the width to the line plot.  </p><br>
  <p>
    The entire process of completing this project took about 30 people-hours split up among the three of us. First, we met over Zoom to discuss which dataset and question we wanted to explore through Project 3. Then, David did the initial setup for hosting the webpage on Github and loading the dataset to our Svelte project. He also created rough line and bar plots to give a starting point. He also worked on setting the search box for different countries and interactivity between the line and bar plot on mouse hovering to further explore each country in a specific year. Nessa helped with the line plot: fine-tuning the aesthetics, creating the button for removing countries, and implementing the constraint to only view 5 countries at a time. Alaa dealt with missing data and NaN values in the bargraph, like Tunisia, and dealth with the bars covering the labels in some countries, like Iceland. She also worked on making the colors of the bars and lines from both graphs match, added the title, axes, and labels to the bargraph. 
</div>

<style>
  @import url('https://urldefense.com/v3/__https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap__;!!Mih3wA!Aycbzsnv_MBp12VkXdbhGtgaYZ57wTbfZe5Y_cuB4d9pUNWtjLVVNWxxVm9mqNc6X1xOKdUhmMJHBu6Ip5rM$ ');
  :root {
    --color-bg: #ffffff;
    --color-outline: #c2c2c2;
    --color-shadow: hsl(0, 0%, 0%, 0.1);
    --color-text: #3f4252;
    --color-bg-1: hsla(0, 0%, 0%, 0.2);
    --color-shadow-1: hsl(0, 0%, 96%);
  }

  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    text-align: center;
    font-family: 'Inria Sans', sans-serif;
    font-weight: 200;
    line-height: 2;
    font-size: 10px;
    color: var(--color-text);
  }

  h1 {
    font-size: 3em;
    font-weight: bolder;
    line-height: 2;
  }
  
  .writeup {
    font-family: 'Inria Sans', sans-serif;
  }

  .dropdown {
    position: relative;
    display: inline-block;
  }

  .dropdown-item:hover {
      background-color: #bce7f4;
  }

  .dropdown-content {
    position: fixed;
    background-color: #f9f9f9;
    box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.1);
    z-index: 1;
    display: block;
    max-height: 400px;
    overflow-y: auto;
    font-size: 15px;
  }

  .dropdown-content div {
    padding: 8px 10px;
    line-height: 1.4;
    cursor: pointer;
  }

  .selected-countries {
    display: flex;
    flex-wrap: wrap;
    margin-top: 10px;
  }

  .selected-country {
    background-color: #f2f2f2;
    padding: 5px 5px;
    margin: 5px;
    position: relative;
    display: flex;
  }

  .done {
    width: 18px;
    height: 18px;
    margin: 0px 5px 0px 0px;
    background-color: white;
    border: 2px solid var(--color-outline);
    appearance: none;
    -webkit-appearance: none;
    outline: none;
    cursor: pointer;
    position: relative;
  }

  .done:before,
  .done:after {
      content: "";
      position: absolute;
      top: 50%;
      left: 0;
      width: 100%;
      height: 2px;
      background-color: var(--color-outline);
  }

  .done:before {
      transform: rotate(45deg);
  }

  .done:after {
      transform: rotate(-45deg);
  }

  .container {
    display: grid;
    grid-template-columns: 7fr 3fr;
  }
</style>