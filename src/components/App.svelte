
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
<p>Exploring the Interplay Between Shower Frequency, Clothes Bought Monthly, and Carbon Emissions</p>
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
<h3 style="text-align: center; font-size: 24px;">Write-Up</h3>

<p>
    Through this visualization, we are exploring how carbon emissions are affected by the monthly 
    distance of a vehicle, while also looking at how shower frequency and number of clothes bought
    monthly have an effect on carbon emission. To explore this, we decided to create an interactive 
    bubble chart with montly distance of a vehicle in kilometers in the x-axis and the carbon 
    emission in the y-axis. We decided to create a bubble chart because it effectively visualized 
    all the variables we wanted to explore.
</p>
 
<p>    
    Additionally, the size of the bubbles change based on the number of clothes bought monthly. 
    To show the size change, we created a tooltip where hovering over a bubble on the chart will 
    provide numerical data on carbon emission, clothes bought monthly, and the number of hours the 
    internet was used daily. 
</p>

<p>    
    In order to represent shower frequency we changed the colors of the bubbles in the chart to 
    represent the different categories of shower frequency. We used bright, divergent colors so 
    that each category could be easily distinguised in the visualization. Additionally, we created 
    an interactive element with the legend where after hovering over the different categories in the 
    legend you can see a different visualization that only portrays bubbles from that shower frequency. 
    This allows viewers to focus on specfic subsets of the data and analyze how the visualization may 
    or may not change depending shower frequency.
</p>

<p>
    Some alternative categories that we considered while creating this visualization were vehicle
    type, diet, and energy source. However, we were more interested in looking what role shower 
    frequency could play on carbon emission. In addition, we considered using different chart 
    types like a bar chart or a heatmap; however a bubble chart allowed us to convey more 
    information and was more suitable for this dataset.
</p>

<p>
    The work for creating this visualization was split evenly among all the team members. We each 
    worked on all parts of the visualization. We met often through zoom to work together on 
    figuring out which dataset and what type of visualization we wanted to create. Then one team
    member worked on the creating the base bubble chart, while the other worked on creating the 
    interactive elements for the visualization. Then together the whole team worked on ensuring 
    the interaction worked well, and that the chart conveyed all the information effectively. 
    Together as a team, we spent around 30 hours developing the application. The most time-
    consuming aspectes setting up the github page, desiging the interactive elements, and ensuring 
    the chart was responsive and intuitive. Overall, each team member worked on every part of the 
    visualization together to create this interactive graph. 
</p>

</body>
