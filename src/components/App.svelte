<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import * as d3 from 'd3';
  import Consumption from './Consumption.svelte';

  let data = [];
  let inputText = '';
  let uniqueCountries = [];
  let selectedCountries = [];
  let dropdownOpen = false;
  const continentList = ['North America', 'South America', 'Europe', 'Africa', 'Asia', 'Oceania', 'World'];

  onMount(async () => {
    const res = await fetch(
      'https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.csv',
    );
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
    // check if the country is already selected
    if (!selectedCountries.includes(country) && (selectedCountries.length < 5)) {
      // add it to the selected countries array
      selectedCountries = [...selectedCountries, country];
      inputText = '';  // reset the query
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

</script>

<main on:click={closeDropdown}>
  <h1>Primary Energy Consumption per Capita Trends</h1>
  <div class="dropdown" on:click={toggleDropdown}>
    <input type="text" placeholder="Search a country" on:input={handleInput} bind:value={inputText}>
    <div class="dropdown-content" style="display: {dropdownOpen ? 'block' : 'none'}">
      {#if inputText !== ''}
        {#each uniqueCountries.filter(country => country.toLowerCase().includes(inputText) && !selectedCountries.includes(country)) as country}
          <div on:click={() => handleCountrySelect(country)}>{country}</div>
        {/each}
      {/if}
    </div>
  </div>

  <div class="selected-countries">
    {#each selectedCountries as country}
      <div class="selected-country" style="color: {getColorClass(country)}" on:click={() => removeSelectedCountry(country)}>{country}</div>
    {/each}
  </div>

  <Consumption {data} {selectedCountries} />
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Protest+Strike');

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
    font-family: 'Protest Strike', protest strike;
    font-weight: 300;
    line-height: 2;
    font-size: 24px;
    color: var(--color-text);
  }

  h1 {
    font-size: 2em;
    font-weight: 300;
    line-height: 2;
  }

  .dropdown {
    position: relative;
    display: inline-block;
  }

  .dropdown-content {
    position: absolute;
    background-color: #f9f9f9;
    min-width: 160px;
    box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
    z-index: 1;
  }

  .dropdown-content div {
    padding: 12px 16px;
    cursor: pointer;
  }

  .selected-countries {
    display: flex;
    flex-wrap: wrap;
    margin-top: 10px;
  }

  .selected-country {
    background-color: #f0f0f0;
    padding: 5px 10px;
    margin: 5px;
    cursor: pointer;
  }
</style>