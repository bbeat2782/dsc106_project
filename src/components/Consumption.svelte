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

  const line_color = d3.scaleOrdinal()
			.range(["#648FFF", "#785EF0", "#DC267F", "#FE6100" ,"#FFB000"]);

  // Placeholders for the axis elements.
  let gx;
  let gy;
  let svg;

  $: maxPrimConsPerCapita = 0
  $: worldData = data.filter(d => d.country === 'World' && d.prim_cons_per_capita !== 0);
  $: countryData = selectedCountries.map(country => {
    return {
        country: country,
        data: data.filter(d => d.country === country && d.prim_cons_per_capita !== 0)
    };
  })

  $: if(selectedCountries) {
    maxPrimConsPerCapita = 0;
  }


  $: countryData.forEach(country => {
    country.data.forEach(d => {
        if (d.prim_cons_per_capita > maxPrimConsPerCapita) {
            maxPrimConsPerCapita = d.prim_cons_per_capita;
        }
    });
  });

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
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(y)
    .tickSize(-(width-marginRight-marginLeft))
    .tickFormat(d3.format(".2s")))
    .call(g => g.select(".domain").remove())
    // grid lines
    .call((g) =>
      g
        .selectAll('.tick line')
        .attr('stroke-opacity', 0.1),
    );

  // Generate line path
  $: line = d3.line()
    .x(d => x(d.year))
    .y(d => y(d.prim_cons_per_capita));

  $: linePath = line(worldData);

  $: linesCountry = countryData.map(c => line(c.data));


  // For tool tips
  const bisect = d3.bisector((d) => d.year).center;
  let tooltipPt = null;
  function onPointerMove(event) {
    const i = bisect(worldData, x.invert(d3.pointer(event)[0]));
    tooltipPt = worldData[i];
  }

  function onPointerLeave(event) {
    tooltipPt = null;
  }

  $: d3.select(svg)
    .on('pointerenter pointermove', onPointerMove)
    .on('pointerleave', onPointerLeave);
</script>

<div class="consumption-plot">
    <svg
      bind:this={svg}
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
              Primary Energy Consumption per Capita (kWh per person)
          </text>
      </g>
      <path d={linePath} fill="none" stroke="black" stroke-width="2" stroke-dasharray="5,5,5" />
      {#each linesCountry as line, i}
          <path d={line} fill="none" stroke={line_color(i)} stroke-width="2" />
      {/each}

      <!-- tooltip -->
      {#if tooltipPt}
        <g transform="translate({x(tooltipPt.year)},{y(tooltipPt.prim_cons_per_capita) + 20})">
          <rect x="-7" y="-7" width="120" height="54" fill="white" stroke="black" />
          <!-- Adjust the translation to center the box vertically -->
          <g transform="translate(0, 5)">
              <text x="0" y="0" font-size="10" text-anchor="left" font-weight="bold">{tooltipPt.country} - {tooltipPt.year}</text>
              <text x="0" y="12" font-size="10" text-anchor="left">Renewables: {tooltipPt.renewables_consumption}</text>
              <text x="0" y="24" font-size="10" text-anchor="left">Nuclear: {tooltipPt.nuclear_consumption}</text>
              <text x="0" y="36" font-size="10" text-anchor="left">Fossil fuel: {tooltipPt.fossil_fuel_consumption}</text>
          </g>
        </g>
      {/if}
    </svg>
</div>
