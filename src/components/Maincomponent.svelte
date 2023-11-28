<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	export let activeGame;
	export let metaScore = null;
    export let graphData = [];
    export let showLess = true;
    export let graphDataLess = [];
    export let graphDataMore = [];

	async function fetchApiData(url) {
		const response = await fetch(url);
		return await response.json();
	}

    function makeBarChart() {
        		// Afmetingen en marges van de grafiek
		const margin = { top: 20, right: 30, bottom: 40, left: 90 },
			width = 460 - margin.left - margin.right,
			height = 400 - margin.top - margin.bottom;

		// SVG object toevoegen
		const svg = d3
			.select('#bar-chart')
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left}, ${margin.top})`);

		// X axis (Horizontale As)
		const x = d3
			.scaleLinear()
			.domain([0, 100]) // Max prijs is 100 / moet nog aanpassen
			.range([0, width]);
		svg.append('g').attr('transform', `translate(0, ${height})`).call(d3.axisBottom(x));

		// Y axis (Verticale as)
		const y = d3
			.scaleBand()
			.range([0, height])
			.domain(graphData.map((d) => d.storeName))
			.padding(0.1);
		svg.append('g').call(d3.axisLeft(y));

		// Staafdiagram toevoegen
		svg
			.selectAll('rect')
			.data(graphData)
			.join('rect')
			.attr('x', x(0))
			.attr('y', (d) => y(d.storeName))
			.attr('width', (d) => x(d.price))
			.attr('height', y.bandwidth())
			.attr('fill', '#69b3a2');

		// Tekstlabels toevoegen
		svg
			.selectAll('.label')
			.data(graphData)
			.enter()
			.append('text')
			.attr('class', 'label')
			.attr('x', (d) => x(d.price) + 5) // ruimte van de prijs af
			.attr('y', (d) => y(d.storeName) + y.bandwidth() / 2 + 5)
			.text((d) => `€${d.price.toFixed(2)}`) // De Prijs
			.attr('text-anchor', 'start') // Tekst x-positie
			.style('fill', 'black') // Kleur van de tekst
			.style('font-size', '12px'); // Grootte van de tekst
    };

	onMount(async () => {
		console.log(activeGame);
		// API URLs
		const dealsUrl = `https://www.cheapshark.com/api/1.0/deals?title=${activeGame.internalName}&exact`;
		const storesUrl = 'https://www.cheapshark.com/api/1.0/stores';

		// Fetch API data
		const dealsData = await fetchApiData(dealsUrl);
		const storesData = await fetchApiData(storesUrl);

		// Itereren door elk 'deal' object in de dealsData array
		for (let deal of dealsData) {
			// Zoekt naar de winkel in storesData die overeenkomt met de storeID van de huidige deal
			const store = storesData.find((s) => s.storeID === deal.storeID);
			// Controleert of de overeenkomende winkel is gevonden
			if (store) {
				graphData.push({
					storeName: store.storeName,
					price: parseFloat(deal.salePrice),
					score: deal.metacriticScore
				}); // Voeg een object met winkelnaam en dealprijs aan de grafiek
				// parseFLoat is een string omzet in een floating-point getal (een getal met decimalen) om de price in een nieuw opject te zetten
			}
		}

		graphData = Object.values(
			graphData.reduce((unique, o) => {
				if (!unique[o.storeName]) unique[o.storeName] = o;

				return unique;
			}, {})
		);

		graphDataLess = graphData.slice(0, 5);
        graphDataMore = graphData.slice(0, 20);
        graphData = graphDataLess;

            makeBarChart();

		// set the dimensions and margins of the graph
		const widthDonot = 450,
			heightDonot = 450,
			marginDonot = 40;

		// The radius of the pieplot is half the width or half the height (smallest one). I subtract a bit of margin.
		const radius = Math.min(widthDonot, heightDonot) / 2 - marginDonot;

		// append the svg object to the div called 'my_dataviz'
		const svgDonot = d3
			.select('#my_dataviz')
			.append('svg')
			.attr('width', widthDonot)
			.attr('height', heightDonot)
			.append('g')
			.attr('transform', `translate(${widthDonot / 2},${heightDonot / 2})`);

		// Create dummy data
		const data = { a: 100 - Number(graphData[0].score), b: 100 };
		metaScore = graphData[0].score;

		// set the color scale
		const color = d3.scaleOrdinal().range(['#98abc5', '#8a89a6', '#7b6888', '#6b486b', '#a05d56']);

		// Compute the position of each group on the pie:
		const pie = d3.pie().value((d) => d[1]);

		const data_ready = pie(Object.entries(data));

		// Build the pie chart: Basically, each part of the pie is a path that we build using the arc function.
		svgDonot
			.selectAll('whatever')
			.data(data_ready)
			.join('path')
			.attr(
				'd',
				d3
					.arc()
					.innerRadius(100) // This is the size of the donut hole
					.outerRadius(radius)
			)
			.attr('fill', (d) => color(d.data[0]))
			.attr('stroke', 'black')
			.style('stroke-width', '2px')
			.style('opacity', 0.7);
	});

    function onClickShowMore() {
		if (showLess) {
            showLess = false;
            graphData = graphDataMore;
        } else {
            showLess = true;
            graphData = graphDataLess;
        }
        document.getElementById('bar-chart').innerHTML = '';
        makeBarChart();
        console.log(graphDataMore, graphDataLess, graphData);
	}
</script>

<div id="bar-chart" />
<button on:click={() => onClickShowMore()}>show {showLess ? 'more' : 'less'}</button>

{#if metaScore}
	<h3>Metascore: {metaScore}</h3>
{/if}
<div id="my_dataviz" />
