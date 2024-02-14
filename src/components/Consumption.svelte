<script>
  import * as d3 from 'd3';
  import { getAllContexts, getContext, createEventDispatcher } from 'svelte';


  export let data;
  export let selectedCountries;

  let countryData = {};
  const width = 928;
  const height = 512;
  const marginTop = 20;
  const marginRight = 0;
  const marginBottom = 30;
  const marginLeft = 40;
  const dispatch = createEventDispatcher();

  const line_color = d3.scaleOrdinal()
			.range(["#648FFF", "#FE6100", "#DC267F", "#FFB000" ,"#785EF0"]);

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

  $: combined = [ ...countryData, {country: 'World', data: data.filter(d => d.country === 'World' && d.prim_cons_per_capita !== 0)}];
  // For tool tips
  const bisect = d3.bisector((d) => d.year).center;
  let tooltipPt = null;
  let debounceTimer;

  function onPointerMove(event) {
    const mouseX = d3.pointer(event)[0];
    const mouseY = d3.pointer(event)[1];
    let closestData = null;
    let closestDistance = Infinity;

    combined.forEach(country => {
      country.data.forEach(d => {
        const xValue = x(d.year);
        const yValue = y(d.prim_cons_per_capita);
        const distance = Math.sqrt((xValue - mouseX) ** 2 + (yValue - mouseY) ** 2);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestData = { ...d, country: country.country };
        }
      });
    });

    tooltipPt = closestData;
    sendDataToApp(tooltipPt);
    updateTooltip(tooltipPt);
  }

  function onPointerLeave(event) {
    tooltipPt = null;
  }

  function updateTooltip(tooltipPt) {
      console.log(tooltipPt.year);
      const tooltipX = x(tooltipPt.year);
      const tooltipY = y(tooltipPt.prim_cons_per_capita);
      let circleColor;
      if (tooltipPt.country === 'World') {
        circleColor = "black";
      } else {
        const lineColorIndex = selectedCountries.indexOf(tooltipPt.country);
        circleColor = line_color(lineColorIndex);
      }

      d3.select(".consumption-plot svg").selectAll(".tooltip-circle").remove(); // Remove previous circles
      d3.select(".consumption-plot svg")
          .append("circle")
          .attr("class", "tooltip-circle")
          .attr("cx", tooltipX)
          .attr("cy", tooltipY)
          .attr("r", 4)
          .attr("fill", circleColor);
    }

  $: d3.select(svg)
    .on('pointerenter pointermove', onPointerMove)
    .on('pointerleave', onPointerLeave);

  function sendDataToApp() {
    dispatch('dataUpdate', tooltipPt);
  }
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

    </svg>
</div>
