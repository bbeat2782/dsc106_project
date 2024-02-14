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
  <h1>Primary Energy Consumption per Capita Trends Across Countries</h1>
  <div class="dropdown" on:click={toggleDropdown}>
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