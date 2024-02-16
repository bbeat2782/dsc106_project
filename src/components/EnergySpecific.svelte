<script>
  import * as d3 from 'd3';

  export let tooltipPt;
  export let selectedCountries;

  let svg;
  let gx;
  let gy;
  
  const width = 400;
  const height = 512;
  const marginTop = 40;
  const marginRight = 30;
  const marginBottom = 40;
  const marginLeft = 100;

  $: barData
  $: if (tooltipPt) {
    barData = [
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
  }

  // Get percentages for each category. If no value, set to 0
  $: barData = barData.map(d => ({
    name: d.name,
    value: d.value == 0 ? 0 : d.value / barData.reduce((acc, d) => acc + d.value, 0)
  }));

  $: barData.sort((a, b) => b.value - a.value);

  $: yScale = d3.scaleBand()
    .domain(barData.map(d => d.name))
    .range([marginTop, height - marginBottom])
    .padding(0.1);

  $: barHeight = yScale.bandwidth();

  $: xScale = d3.scaleLinear()
    .domain([0, 1])
    .range([marginLeft, width - marginRight]);

  function getColorClass(country) {
      if (country === "World") {
        return "#000000";
      } else {
        const index = selectedCountries.indexOf(country);
        const colors = ["#648FFF", "#FE6100", "#DC267F", "#FFB000" ,"#785EF0"];
        return colors[index % colors.length];
      }
  }

  function createBars() {
      const bars = d3.select(svg)
        .selectAll("rect")
        .data(barData);
      let col = getColorClass(tooltipPt.country);

      // Add bars for categories
      bars.exit().remove();
      bars.enter()
          .append("rect")
          .attr("fill", col)
          .attr("y", d => yScale(d.name))
          .attr("height", barHeight)
          .attr("x", marginLeft)
          .attr("width", 0)
          .merge(bars)
          .transition()
          .duration(100)
          .attr("fill", d => getColorClass(tooltipPt.country))
          .attr("width", d => Math.abs(xScale(d.value) - xScale(0)))
          .attr("x", d => Math.min(xScale(d.value), xScale(0)));

      // Add labels for categories
      const labels = d3.select(svg)
        .selectAll(".label")
        .data(barData);
      labels.exit().remove();
      labels.enter()
        .append("text")
        .attr("class", "label")
        .merge(labels)
        .attr("x", marginLeft - 5)
        .attr("y", d => yScale(d.name) + barHeight / 2)
        .attr("text-anchor", "end")
        .attr("alignment-baseline", "middle")
        .attr("font-size", "12px")
        .text(d => d.name);

      // Add horizontal gridlines
      const gridlines = d3.select(svg)
          .selectAll(".gridline")
          .data(yScale.domain());
      gridlines.enter()
          .append("line")
          .attr("class", "gridline")
          .merge(gridlines)
          .attr("x1", marginLeft)
          .attr("x2", width - marginRight*1.6)
          .attr("y1", d => yScale(d) + barHeight / 2)
          .attr("y2", d => yScale(d) + barHeight / 2)
          .attr("stroke", "#ccc")
          .attr("stroke-dasharray", "3,3");

      // Add percentages right side of each category
      const percentages = d3.select(svg)
          .selectAll(".percentage")
          .data(barData);
      percentages.exit().remove();
      percentages.enter()
        .append("text")
        .attr("class", "percentage")
        .merge(percentages)
        .attr("x", d => xScale(0.95))
        .attr("y", d => yScale(d.name) + barHeight / 2)
        .attr("text-anchor", "start")
        .attr("alignment-baseline", "middle")
        .attr("font-size", "12px")
        .attr("fill", "black")
        .text(d => `${(d.value * 100).toFixed(2)}%`);
  }

  function sourceNotAvailable() {
    const bars = d3.select(svg)
        .selectAll("rect")
        .remove();
  }

  $: if (isNaN(barData[0].value)) {
    // Error handling for cases where there is no value for each category
    sourceNotAvailable();
  } else {
    createBars();
  }

  // Setting the intial data of World-2020 to be shown
  $: try {
      if (!tooltipPt.country) {
          tooltipPt = {
              country: "World",
              year: "2020",
              prim_cons_per_capita: 0
          };
      }
  } catch (error) {
    console.log('this');
    tooltipPt = {
        country: "World",
        year: "2020",
        prim_cons_per_capita: 0
    };
  }

  // Formatting number so that it's easier to read
  function formattedNumber(number) {
      let formattedNumber = number.toString();
      formattedNumber = formattedNumber.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      return formattedNumber;
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
    <g>
      <text
      x={marginLeft}
      y={marginTop / 2}
      text-anchor="left"
      alignment-baseline="middle"
      font-size="15px"
    >
      Energy Consumption Source %
      <tspan dy="1.45em" x={marginLeft} font-size="13px">{tooltipPt.country} - {tooltipPt.year}</tspan>
      <tspan dx="5px"></tspan>
      <tspan font-size="10px">({formattedNumber(tooltipPt.prim_cons_per_capita)} kWh)</tspan>
    </text>
    </g>
    <text x={marginLeft/2}
    y={height - marginBottom/2}
    >
      Note: Bars are not shown when there is no data
    </text>
  </svg>
</div>
