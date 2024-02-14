<script>
  import * as d3 from 'd3';

  export let tooltipPt;
  export let selectedCountries;

  let svg;
  let gx;
  let gy;
  

  const width = 400;
  const height = 512;
  const marginTop = 40; // Adjusted margin top to accommodate labels
  const marginRight = 30;
  const marginBottom = 40; // Increased margin bottom to ensure space for labels
  const marginLeft = 100; // Adjusted margin left to create space for labels and axes

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

  // $: console.log(barData);
  $: barData = barData.map(d => ({
    name: d.name,
    value: d.value == 0 ? 0 : d.value / barData.reduce((acc, d) => acc + d.value, 0)
  }));
  // $: console.log(barData);

  $: barData.sort((a, b) => b.value - a.value);

  $: yScale = d3.scaleBand()
    .domain(barData.map(d => d.name))
    .range([marginTop, height - marginBottom])
    .padding(0.1);

  $: barHeight = yScale.bandwidth(); // Fixed height for bars

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

      const labels = d3.select(svg)
        .selectAll(".label")
        .data(barData);

      labels.exit().remove();

      labels.enter()
        .append("text")
        .attr("class", "label")
        .merge(labels)
        .attr("x", marginLeft - 5)
        .attr("y", d => yScale(d.name) + barHeight / 2) // Position labels in the middle of the bar
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
          .attr("x2", width - marginRight*3)
          .attr("y1", d => yScale(d) + barHeight / 2)
          .attr("y2", d => yScale(d) + barHeight / 2)
          .attr("stroke", "#ccc") // Adjust the color of the gridlines
          .attr("stroke-dasharray", "3,3"); // Add dashed lines if desired

      gridlines.exit().remove();

      // Add percentages
      const percentages = d3.select(svg)
          .selectAll(".percentage")
          .data(barData);

      percentages.exit().remove();

      percentages.enter()
        .append("text")
        .attr("class", "percentage")
        .merge(percentages)
        .attr("x", d => xScale(0.8)) // Set x-coordinate to the maximum value of xScale
        .attr("y", d => yScale(d.name) + barHeight / 2) // Adjust y position to center the text vertically
        .attr("text-anchor", "start")
        .attr("alignment-baseline", "middle")
        .attr("font-size", "12px")
        .attr("fill", "black") // Set text color
        .text(d => `${(d.value * 100).toFixed(2)}%`);

      // percentages.enter()
      //   .append("text")
      //   .attr("class", "percentage")
      //   .merge(percentages)
      //   .attr("x", d => xScale(d.value) + 5) // Adjust the positioning as needed
      //   .attr("y", d => yScale(d.name) + barHeight / 2) // Adjust y position to center the text vertically
      //   .attr("text-anchor", "start")
      //   .attr("alignment-baseline", "middle")
      //   .attr("font-size", "12px")
      //   .attr("fill", "black") // Set text color
      //   .text(d => `${(d.value * 100).toFixed(2)}%`);

  }

  function sourceNotAvailable() {
    const bars = d3.select(svg)
        .selectAll("rect")
        .remove();
  }

  // $: color = getColorClass(tooltipPt.country);
  $: if (isNaN(barData[0].value)) {
    // add something here
    sourceNotAvailable();
  } else {
    createBars();
  }

  $: try {
      if (!tooltipPt.country) {
          tooltipPt = {
              country: "World",
              year: "2020"
          };
      }
  } catch (error) {
    console.log('this');
    tooltipPt = {
        country: "World",
        year: "2020"
    };
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
      x={width / 2}
      y={marginTop / 2}
      text-anchor="middle"
      alignment-baseline="middle"
      font-size="15px"
    >
      Energy Consumption Source %
      <tspan dy="1.35em" x={width / 2} font-size="13px">{tooltipPt.country} - {tooltipPt.year}</tspan>
    </text>
      <!-- Bars will be rendered here -->
    </g>
    <text x={marginLeft/2}
    y={height - marginBottom/2}
    >
      Note: Bars are not shown when there is no data
    </text>
  </svg>
</div>
