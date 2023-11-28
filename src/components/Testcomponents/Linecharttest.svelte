<!-- Dit was mijn voor mijn historical data maar is niet gelukt. Source: https://d3-graph-gallery.com/graph/line_basic.html -->
<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    onMount(() => {
        // set the dimensions and margins of the graph
        const margin = {top: 10, right: 30, bottom: 30, left: 60},
            width = 460 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;

        // select the svg area
        const svg = d3.select("#line-chart")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        // Read the data and draw the chart
        d3.csv("https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/3_TwoNumOrdered_comma.csv",
            function(d){
                return { date : d3.timeParse("%Y-%m-%d")(d.date), value : d.value }
            }).then(function(data) {
            // X axis
            const x = d3.scaleTime()
                .domain(d3.extent(data, function(d) { return d.date; }))
                .range([ 0, width ]);
            svg.append("g")
                .attr("transform", `translate(0, ${height})`)
                .call(d3.axisBottom(x));

            // Y axis
            const y = d3.scaleLinear()
                .domain([0, d3.max(data, function(d) { return +d.value; })])
                .range([ height, 0 ]);
            svg.append("g")
                .call(d3.axisLeft(y));

            // Line
            svg.append("path")
                .datum(data)
                .attr("fill", "none")
                .attr("stroke", "steelblue")
                .attr("stroke-width", 1.5)
                .attr("d", d3.line()
                    .x(function(d) { return x(d.date) })
                    .y(function(d) { return y(d.value) })
                )
        });
    });
</script>

<svg id="line-chart"></svg>
