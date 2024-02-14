<script>
  import * as d3 from 'd3';

  export let tooltipPt;
  export let selectedCountries;

  let svg;
  let gx;
  let gy;
  // const width = window.innerWidth*0.5;
  // const height = window.innerHeight*0.5;
  const width = 928;
  const height = 512;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 30;
  const marginLeft = 40;

  // $: worldData = data.filter(d => d.country === 'World' && d.prim_cons_per_capita !== 0);
  // $: countryData = selectedCountries.map(country => {
  //   return {
  //       country: country,
  //       data: data.filter(d => d.country === country && d.prim_cons_per_capita !== 0)
  //   };
  // })

  $: barData = [
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
  // $: console.log(tooltipPt.country);
  $: barData = barData.map(d => ({
    name: d.name,
    value: isNaN(d.value) ? 0 : d.value
  }));
  $: barData.sort((a, b) => b.value - a.value);
  $: barData = barData.map(d => ({
    name: d.name,
    value: d.value / (barData.reduce((acc, d) => acc + d.value, 0)) * 100
  }))

  $: xScale = d3.scaleBand()
    .domain(barData.map(d => d.name))
    .range([marginLeft, width - marginRight])
    .padding(0.1);

  $: yScale = d3.scaleLinear()
    .domain([0, 100])
    .range([height - marginBottom, marginTop]);

  function getColorClass(country) {
      if (country === "World") {
        return "#000000";
      } else {
        const index = selectedCountries.indexOf(country);
        const colors = ["#648FFF", "#FE6100", "#DC267F", "#FFB000" ,"#785EF0"];
        return colors[index % colors.length];
      }
  }

  function createBars(color) {
    // console.log(color);
    const bars = d3.select(svg)
      .selectAll("rect")
      .data(barData);
    
    bars.exit().remove();

    bars.enter()
        .append("rect")
        .attr("fill", color)
        .merge(bars) // Merge new and existing bars
        .attr("x", d => xScale(d.name))
        .attr("width", xScale.bandwidth())
        .attr("y", height - marginBottom) // Start the bars at the bottom
        .attr("height", 0) // Initialize height to 0
        .transition()
        .duration(500)
        .attr("height", d => Math.abs(yScale(d.value) - yScale(0))) // Set the height based on the difference between top and bottom positions
        .attr("y", d => {
            const barHeight = Math.abs(yScale(d.value) - yScale(0));
            return height - marginBottom - barHeight;
        });

    const labels = d3.select(svg)
      .selectAll(".label")
      .data(barData);

    labels.exit().remove();

    labels.enter()
      .append("text")
      .attr("class", "label")
      .merge(labels)
      .attr("x", d => xScale(d.name) + xScale.bandwidth() / 2)
      .attr("y", height - marginBottom + 15)
      .attr("text-anchor", "middle")
      .attr("font-size", "12px")
      .text(d => d.name);
  }

  $: color = getColorClass(tooltipPt.country);
  $: if (isNaN(barData[0].value)) {
    // add something here
  } else {
    createBars(color);
  }

</script>



<div class="energyspecific-plot">
  <svg
    bind:this={svg}
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
  >
    <g transform={`translate(${marginLeft},${marginTop})`}>
      <!-- Bars will be rendered here -->
    </g>

  </svg>
</div>
