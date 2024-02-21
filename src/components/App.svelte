
<head>
<style>
  body {
    background-color: #cccccc;
     /* Center aligning the text */
     text-align: right;
  }
  h1 {
    color: black; /* Changing text color */
    text-align: center;
  }
  h2 {
    text-align: left;
    font-size: 15px;
    font-weight: normal;

  }
  p {
    color: black; /* Changing text color */
    font-size: 20px;
    text-align: center;
  }

  h3 {
    text-align: left;
  }
  h4 {
    text-align: left;
  }


</style>
</head>
<body>

<h1>Shower Power: Scrubbing the Earth Clean or Washing Away Our Future?</h1>
<p>Exploring the Interplay Between Shower Frequency, Waste Generation, and Carbon Emissions</p>
<h2>Hover Data</h2>
</body>

<script>

    import * as d3 from 'd3';
    
    import { onMount } from 'svelte';

    let energyData = [];
    let emissionData = []

    // onMount(async () => {

    //    const res = await fetch('raw.githubusercontent.com_owid_energy-data_master_owid-energy-data.csv'); 

    //    const csv = await res.text();

    //    energyData = d3.csvParse(csv, d3.autoType)

    //    console.log(energyData);

    // });
    onMount(async () => {

        const res = await fetch('Carbon Emission.csv'); 

        const csv = await res.text();

        emissionData = d3.csvParse(csv, d3.autoType)

        $: console.log(emissionData);
        createBubbleChart();

    });

    function createBubbleChart() {
        // Define SVG dimensions and margins
        const width = 1000;
        const height = 800;
        const margin = { top: 50, right: 50, bottom: 70, left: 70 };

        // Append SVG to the HTML document
        const svg = d3.select("svg")
                      .attr("width", width)
                      .attr("height", height)
                      ;

        // Find maximum values for x and y axes
        const maxX = d3.max(emissionData, d => d["Vehicle Monthly Distance Km"]);
        const maxY = d3.max(emissionData, d => d.CarbonEmission);

        // Create linear scales for x and y axes
        const xScale = d3.scaleLinear()
                         .domain([-500, maxX + 1000])
                         .range([margin.left, width - margin.right]);

        const yScale = d3.scaleLinear()
                         .domain([0, maxY + 1000])
                         .range([height - margin.bottom, margin.top]);

        const radiusScale = d3.scaleSqrt()
                              .domain([0, d3.max(emissionData, d => d["How Many New Clothes Monthly"])])
                              .range([2, 20]); // Adjust as needed

        const categories = Array.from(new Set(emissionData.map(d => d["How Often Shower"])));
        //console.log(categories);

        const colorScale = d3.scaleOrdinal()
                         .domain(categories)
                         .range(["#2F9599", "#E84A5F", "#F7DB4F", "#F26B38", "#355C7D", "#2F9599"]);

        // Add circles for each data point
        const circles = svg.selectAll("circle")
           .data(emissionData)
           .enter()
           .append("circle")
           .attr("cx", d => xScale(d["Vehicle Monthly Distance Km"]))
           .attr("cy", d => yScale(d.CarbonEmission))
           .attr("r", d => radiusScale(d["How Many New Clothes Monthly"]))
           .attr("fill", d => colorScale(d["How Often Shower"]))
           .attr("opacity", 0.7);
        

        // Add labels for x and y axes
        svg.append("g")
           .attr("transform", `translate(0, ${height - margin.bottom})`)
           .call(d3.axisBottom(xScale).tickValues(d3.range(0, maxX + 600, 500)));


        svg.append("g")
           .attr("transform", `translate(${margin.left}, 0)`)
           .call(d3.axisLeft(yScale));

        // Add X axis label
        svg.append("text")
           .attr("text-anchor", "middle")
           .attr("x", width/2)
           .attr("y", height - 10)
           .text("Vehicular Monthly Distance (Km)");

        // Add Y axis label
        svg.append("text")
           .attr("text-anchor", "middle")
           .attr("transform", "rotate(-90)")
           .attr("y", margin.left / 3) // Position at the center of the SVG vertically
           .attr("x", -height / 2) // Position at the center of the SVG horizontally (negative because of rotation)
           .text("Carbon Emission");

        const legend = svg.selectAll(".legend")
            .data(categories)
            .enter().append("g")
            .attr("class", "legend")
            .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; })
            .on("mouseover", function(event, d) { updateVisualization(event, d); })
            .on("click", function(event, d) { updateVisualization(event, d); });

        // Add colored rectangles to legend
        legend.append("rect")
            .attr("x", width - margin.right - 18)
            .attr("width", 18)
            .attr("height", 18)
            .style("fill", colorScale);

        // Add text to legend
        legend.append("text")
            .attr("x", width - margin.right - 24)
            .attr("y", 9)
            .attr("dy", ".35em")
            .style("text-anchor", "end")
            .text(function(d) {
            if (d === 'Less frequently') {
                return capitalizeFirstLetter("Less Than Once a Day");
            } else if (d === 'More frequently') {
                return capitalizeFirstLetter("More Than Twice a Day");
            } else {
                return capitalizeFirstLetter(d === null ? "Unknown" : d);
            } }
            );

        // Add tooltip
        const tooltip = d3.select("h2").append("div")
                          .attr("class", "tooltip")
                          .style("opacity", 0);

        svg.selectAll("circle")
           .on("mouseover", function(event, d) {
               tooltip.transition()
                      .duration(200)
                      .style("opacity", .9);
               tooltip.html(`Carbon Emission: ${d.CarbonEmission}<br>Internet Daily Hour: ${d["How Long Internet Daily Hour"]}<br>Clothes Monthly: ${d["How Many New Clothes Monthly"]}`)
                      .style("right", (event.pageX) + "px")
                      .style("top", (event.pageY-28) + "px");
           })
           .on("mouseout", function(d) {
               tooltip.transition()
                      .duration(500)
                      .style("opacity", 0);
           });

        function updateVisualization(event, category) {
            const selectedCategory = category === "Unknown" ? null : category;
            circles.attr("display", function(d) {
                return d["How Often Shower"] === selectedCategory ? "block" : "none";
            });
        }
        
        function resetVisualization() {
            d3.selectAll("circle").attr("display", "block");
        }

        function capitalizeFirstLetter(string) {
            return string.charAt(0).toUpperCase() + string.slice(1);
        }
    }

    
</script>
<svg style="display:block;margin:auto;"></svg>

<body>
<h3>Design Rationale</h3>

<h4>Visual Encodings:</h4>
<p>
    <strong>Bubble Size:</strong> You've chosen to represent the magnitude of waste generation (specifically, "How Many New Clothes Monthly") using bubble size. This is an effective choice as it allows viewers to quickly grasp the relative amounts of waste generated by each data point.
</p>
<p>
    <strong>Bubble Color:</strong> The color of the bubbles represents shower frequency. Using color for categorical data is a common and intuitive choice, making it easy for viewers to distinguish between different shower frequencies.
</p>
<p>
    <strong>X and Y Axes:</strong> You've opted for a scatter plot with vehicular monthly distance on the x-axis and carbon emissions on the y-axis. This choice allows viewers to easily see the relationship between these two variables.
</p>

<h4>Interaction Techniques:</h4>
<p>
    <strong>Tooltip:</strong> Implementing a tooltip provides additional information about each data point when users hover over them. This enhances interactivity and helps users understand the specific values associated with each bubble.
</p>
<p>
    <strong>Legend Interaction:</strong> Clicking on legend items to filter the visualization based on shower frequency is a useful interaction technique. It allows users to focus on specific subsets of the data and analyze them in more detail.
</p>

<h4>Design Rationale:</h4>
<p>
    <strong>Bubble Chart:</strong> The bubble chart is a suitable choice for this dataset because it can effectively visualize three variables: vehicular monthly distance (x-axis), carbon emissions (y-axis), and waste generation (bubble size). This allows for a comprehensive understanding of the relationships between these variables.
</p>
<p>
    <strong>Color Scheme:</strong> The chosen color scheme for shower frequency (shades of blue and red) is visually appealing and intuitive. It provides clear differentiation between different shower frequencies without being overwhelming.
</p>
<p>
    <strong>Tooltip Positioning:</strong> You've positioned the tooltip adjacent to the bubbles, making it easier for users to associate the tooltip content with the corresponding data point. Placing it to the right of the cursor ensures it doesn't obscure the visualization.
</p>

<h4>Alternatives Considered:</h4>
<p>
    <strong>For representing waste generation,</strong> alternatives to bubble size could include color intensity or a separate bar chart overlaid on the bubbles. However, bubble size is effective for showing magnitude directly.
</p>
<p>
    <strong>Different chart types like a bar chart</strong> or a heatmap could have been considered for showing the relationship between vehicular monthly distance and carbon emissions, but a scatter plot is commonly used for this purpose.
</p>
<p>
    <strong>Instead of a legend,</strong> interactive buttons or dropdowns could be used for filtering data based on shower frequency. However, a legend is a straightforward and widely understood method for interacting with categorical data.
</p>

<p>
    In conclusion, your design decisions prioritize clarity, interactivity, and effectiveness in conveying the relationships within the dataset. The chosen visual encodings and interaction techniques align well with the goals of the project and provide users with a rich and informative data exploration experience.
</p>

</body>
