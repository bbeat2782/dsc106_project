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

  function updateTooltipDebounced(tooltipPt) {
      // Clear previous debounce timer
      clearTimeout(debounceTimer);

      if (tooltipPt !== null) {
          updateTooltip(tooltipPt);
          return;
      }

      // Set a new debounce timer to update the tooltip after a short delay
      debounceTimer = setTimeout(() => {
          updateTooltip(tooltipPt);
      }, 10);
  }


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
    updateTooltipDebounced(tooltipPt);
  }

  function onPointerLeave(event) {
    tooltipPt = null;
    updateTooltipDebounced(tooltipPt);
  }

  let tooltip;
  function updateTooltip(tooltipPt) {
    if (!tooltip) {
      // Initialize tooltip for the first time
      tooltip = d3.select(".consumption-plot")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("background-color", "white")
        .style("padding", "5px")
        .style("border", "1px solid black")
        .style("display", "none")
        .style("width", "200px");
    }

    if (tooltipPt) {
      // const tooltipX = x(tooltipPt.year);
      // const tooltipY = y(tooltipPt.prim_cons_per_capita);

      // ADJUSTING TOOLTIP 
      const tooltipWidth = tooltip.node().offsetWidth;
      const tooltipHeight = tooltip.node().offsetHeight;

      let tooltipX = x(tooltipPt.year) + marginLeft;
      let tooltipY = y(tooltipPt.prim_cons_per_capita) + marginTop;

      // Check right
        if (tooltipX + tooltipWidth > width) {
            tooltipX = width - tooltipWidth;
        }


      tooltip.html(`
        <div class="tooltip">
          <b>${tooltipPt.country} - ${tooltipPt.year}</b><br>(${tooltipPt.prim_cons_per_capita} kWh)
          <div class="tooltip-content">
              <div class="barplot"></div>
          </div>
        </div>
      `)
      .style("left", `${tooltipX}px`)
      .style("top", `${tooltipY}px`)
      .style("display", "block")
      .style("line-height", "1.15")
      .style("transform", "translate(-25%, 50%)");

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
          .attr("r", 3.5)
          .attr("fill", circleColor);

      // Draw bar plot
      let barData = [
        { name: "Biofuel", value: tooltipPt.biofuel_consumption },
        { name: "Coal", value: tooltipPt.coal_consumption },
        { name: "Fossil fuel", value: tooltipPt.fossil_fuel_consumption },
        { name: "Gas", value: tooltipPt.gas_consumption },
        { name: "Hydro", value: tooltipPt.hydro_consumption },
        { name: "Low Carbon", value: tooltipPt.low_carbon_consumption },
        { name: "Nuclear", value: tooltipPt.nuclear_consumption },
        { name: "Oil", value: tooltipPt.oil_consumption },
        { name: "Wind", value: tooltipPt.wind_consumption },
        { name: "Solar", value: tooltipPt.solar_consumption }
      ];

      barData.sort((a, b) => b.value - a.value);
      const topThree = barData.slice(0, 3);
      const othersTotal = barData.slice(3).reduce((sum, entry) => sum + entry.value, 0);
      const others = { name: "Others", value: othersTotal };
      barData = topThree.concat(others);
      barData.sort((a, b) => b.value - a.value);
      const total = barData.reduce((acc, d) => acc + d.value, 0); 

      const tooltipContent = tooltip.select(".tooltip-content");
      tooltipContent.selectAll(".barplot").remove();

      tooltip.select(".tooltip")
        .style("font-size", "13px");

      tooltip.select(".tooltip-content")
        .style("font-size", "10px");
      
      const bars = tooltipContent.selectAll(".barplot")
          .data(barData)
          .enter().append("div")
          .attr("class", "barplot")
          .style("display", "flex")
          .style("height", "15px")
          .style("margin-top", "5px")
          .style("position", "relative");

      bars.append("div")
          .text(d => d.name)
          .style("position", "absolute")
          .style("text-align", "left")
          .style("white-space", "nowrap");

      const barBlocks = bars.append("div")
          .style("display", "flex")
          .style("justify-content", "flex-end")
          .style("height", "100%")
          .style("width", "100%")
          .style("position", "relative");

      barBlocks.append("div")
          .style("background-color", "steelblue")
          .style("height", "100%")
          .style("width", d => `${Math.round(d.value / total * 100)}%`)
          .style("margin-right", "22px");

      barBlocks.append("div")
          .style("position", "absolute")
          .style("right", "0")
          .style("top", "0")
          .style("font-size", "0.8em")
          .style("padding-right", "2px")
          .style("line-height", "15px")
          .text(d => `${Math.round(d.value / total * 100)}%`);
    } else {
      tooltip.style("display", "none");
    }
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
      {/if}
    </svg>
</div>
