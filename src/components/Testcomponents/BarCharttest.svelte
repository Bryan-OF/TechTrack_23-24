<!-- Originele barcharts is van https://codepen.io/bryan-of/pen/KKJWyYX deze was mijn eerste test chart -->
<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let barsGroup;
    let labelsGroup;

    onMount(() => {
        const dataSet = [
            { day: "Monday", cars: 20 },
            { day: "Tuesday", cars: 50 },
            { day: "Wednesday", cars: 80 },
            { day: "Thursday", cars: 150 },
            { day: "Friday", cars: 200 },
            { day: "Saturday", cars: 300 }
        ];

        const chartWidth = 700;
        const chartHeight = 200;

        const xScale = d3.scaleLinear()
            .domain([0, d3.max(dataSet, d => d.cars)])
            .range([0, chartWidth]);

        const yScale = d3.scaleBand()
            .domain(dataSet.map(d => d.day))
            .range([0, chartHeight])
            .paddingInner(0.05);

        const bars = d3.select(barsGroup)
            .selectAll("g")
            .data(dataSet)
            .join("g");

        bars.append("rect")
            .attr("height", yScale.bandwidth())
            .attr("width", d => xScale(d.cars))
            .attr("y", d => yScale(d.day));

        bars.append("text")
            .style("dominant-baseline", "middle")
            .attr("y", d => yScale(d.day) + yScale.bandwidth() / 2)
            .attr("x", d => xScale(d.cars) - 5)
            .text(d => d.cars);

        d3.select(labelsGroup)
            .selectAll("text")
            .data(dataSet)
            .join("text")
            .style("dominant-baseline", "middle")
            .attr("y", d => yScale(d.day) + yScale.bandwidth() / 2)
            .text(d => d.day);
    });
</script>

<style>
    :global(text) {
        font-family: sans-serif;
    }

    :global(#bars text) {
        fill: white;
        text-anchor: end;
    }
</style>

<svg width="800" height="300">
    <g bind:this={barsGroup} id="bars" transform="translate(90, 0)">
    </g>
    <g bind:this={labelsGroup} id="labels"></g>
</svg>