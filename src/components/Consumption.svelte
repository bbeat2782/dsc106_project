<script>
  import * as d3 from 'd3';
  import { getContext } from 'svelte';


  export let data;
  export let selectedCountries;

  let countryData = {};

  const width = 928;
  const height = 600;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 30;
  const marginLeft = 40;

  // Placeholders for the axis elements.
  let gx;
  let gy;

  $: maxPrimConsPerCapita = 0
  $: worldData = data.filter(d => d.country === 'World' && d.prim_cons_per_capita !== 0);
  $: countryData = selectedCountries.map(country => {
    return {
        country: country,
        data: data.filter(d => d.country === country && d.prim_cons_per_capita !== 0)
    };
  })

  // NOTE takes too long to draw all countries
  // $: if (selectedCountries.length == 0) {
  //   countryData = data.map(d => {
  //     return {
  //       country: d.country,
  //       data: data.filter(v => v.prim_cons_per_capita !== 0)
  //     }
  //   });
  //   console.log(countryData);
  // }

  $: if(selectedCountries) {
    maxPrimConsPerCapita = 0;
    // reset gridlines TODO
    // d3.select(gy).call((g) => g.selectAll('.tick line').remove());
  }


//   $: maxPrimConsPerCapita = 0;
  $: countryData.forEach(country => {
    country.data.forEach(d => {
        if (d.prim_cons_per_capita > maxPrimConsPerCapita) {
            maxPrimConsPerCapita = d.prim_cons_per_capita;
        }
    });
  });

  // $: console.log(maxPrimConsPerCapita);
  // $: console.log(selectedCountries);

  $: years = worldData.map(d => d.year);
  $: world_primary_energy_cons_per_capita = worldData.map(d => d.prim_cons_per_capita);
  $: maxPrimConsPerCapita = maxPrimConsPerCapita > d3.max(world_primary_energy_cons_per_capita) ? maxPrimConsPerCapita : d3.max(world_primary_energy_cons_per_capita);

  $: x = d3.scaleTime()
    .domain([d3.min(years), d3.max(years)])
    .range([marginLeft, width - marginRight]);


  $: y = d3.scaleLinear()
    .domain([0, maxPrimConsPerCapita])
    .nice()
    .range([height - marginBottom, marginTop]);


  $: d3.select(gx)
    .call(d3.axisBottom(x)
    .ticks(width / 80)
    .tickFormat(d3.format("")));

  
  $: d3.select(gy)
    .call(d3.axisLeft(y)
    // .ticks(null, '+')
    .tickFormat(d3.format(".2s")))
    // grid lines
    .call((g) =>
      g
        .selectAll('.tick line')
        .clone()
        .attr('x2', width - marginRight - marginLeft)
        //.attr('stroke-opacity', (d) => (d === 0 ? 1 : 0.1)),
        .attr('stroke-opacity', 0.1),
    );

  // Generate line path
  $: line = d3.line()
    .x(d => x(d.year))
    .y(d => y(d.prim_cons_per_capita));

  $: linePath = line(worldData);

  $: linesCountry = countryData.map(c => line(c.data));
  $: console.log(linesCountry);
</script>

<div class="consumption-plot">
    <svg
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
  >
    <!-- x-axis -->
    <g bind:this={gx} transform="translate(0,{height - marginBottom})" />
    <!-- y-axis -->
    <g bind:this={gy} transform="translate({marginLeft},0)">
        <text
            x="5"
            y={marginTop}
            dy="0.32em"
            fill="#000"
            font-weight="bold"
            text-anchor="start"
        >
            Primary Energy Consumption per Capita
        </text>
    </g>
    <path d={linePath} fill="none" stroke="blue" stroke-width="2" />
    {#each linesCountry as line, i}
        <path d={line} fill="none" stroke="black" stroke-width="2" />
    {/each}
  </svg>
</div>
