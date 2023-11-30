<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	// Geëxporteerde variabelen voor gebruik buiten het component
	export let activeGame;
	export let metaScore = null;
    export let graphData = []; // Data voor de grafiek
    export let showLess = true; // Status om meer of minder data te tonen
    export let graphDataLess = []; // Data voor de optie "minder tonen"
    export let graphDataMore = []; // Data voor de optie "meer tonen"

	// Functie om API data op te halen
	async function fetchApiData(url) {
		const response = await fetch(url);
		return await response.json();
	}

	// Functie om een staafdiagram te maken
    function makeBarChart() {
        // Instellen van afmetingen en marges voor de grafiek
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

		// X-as (Horizontale as)
		const x = d3
			.scaleLinear()
			.domain([0, 60])
			.range([0, width]);
		svg.append('g').attr('transform', `translate(0, ${height})`).call(d3.axisBottom(x));

		// Y-as (Verticale as)
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
			.attr('x', (d) => x(d.price) + 5) // Ruimte vanaf de prijs
			.attr('y', (d) => y(d.storeName) + y.bandwidth() / 2 + 5)
			.text((d) => `€${d.price.toFixed(2)}`) // De prijs
			.attr('text-anchor', 'start') // Positie van de tekst op de x-as
			.style('fill', 'black') // Kleur van de tekst
			.style('font-size', '12px'); // Grootte van de tekst
    };

	// Zorgt dat de functie wordt uitgevoerd wanneer het component geladen is
	onMount(async () => {
		console.log(activeGame);
		// API URL's
		const dealsUrl = `https://www.cheapshark.com/api/1.0/deals?title=${activeGame.internalName}&exact`;
		const storesUrl = 'https://www.cheapshark.com/api/1.0/stores';

		// Ophalen van API data
		const dealsData = await fetchApiData(dealsUrl);
		const storesData = await fetchApiData(storesUrl);

		// Itereren door elk 'deal' object in de dealsData array
		for (let deal of dealsData) {
			// Zoeken naar de winkel in storesData die overeenkomt met de storeID van de huidige deal
			const store = storesData.find((s) => s.storeID === deal.storeID);
			// Controleren of de overeenkomende winkel gevonden is
			if (store) {
				graphData.push({
					storeName: store.storeName,
					price: parseFloat(deal.salePrice),
					score: deal.metacriticScore
				}); // Voegt een object met winkelnaam en dealprijs toe aan de grafiek
				// parseFloat zet een string om in een floating-point getal (een getal met decimalen) om de prijs in een nieuw object te zetten
			}
		}

		// Maakt een lijst van unieke winkels uit 'graphData'.
		graphData = Object.values(
			// Voegt nieuwe winkel toe als deze nog niet in de lijst staat.
			graphData.reduce((unique, o) => {
				if (!unique[o.storeName]) unique[o.storeName] = o;

				// Geeft de bijgewerkte lijst terug.
				return unique;
			}, {})
		);

		// Kiest de eerste 5 winkels voor een kortere lijst.
		graphDataLess = graphData.slice(0, 5);
		// Kiest de eerste 20 winkels voor een langere lijst.
        graphDataMore = graphData.slice(0, 20);
		// Gebruikt de kortere lijst voor de grafiek.
        graphData = graphDataLess;

		// Maakt de grafiek met de geselecteerde winkels.
        makeBarChart();

		// Instellen van de afmetingen en marges van de grafiek
		const widthDonot = 450,
			heightDonot = 450,
			marginDonot = 40;

		// De radius van de taartgrafiek is de helft van de breedte of hoogte (de kleinste van de twee). Ik trek er een beetje marge van af.
		const radius = Math.min(widthDonot, heightDonot) / 2 - marginDonot;

		// Voeg het SVG object toe aan de div met de naam 'my_dataviz'
		const svgDonot = d3
			.select('#my_dataviz')
			.append('svg')
			.attr('width', widthDonot)
			.attr('height', heightDonot)
			.append('g')
			.attr('transform', `translate(${widthDonot / 2},${heightDonot / 2})`);

		// Maak dummy data
		const data = { a: 100 - Number(graphData[0].score), b: 100 };
		metaScore = graphData[0].score;

		// Instellen van de kleurenschaal
		const color = d3.scaleOrdinal().range(['#98abc5', '#8a89a6', '#7b6888', '#6b486b', '#a05d56']);

		// Bereken de positie van elke groep in de taartgrafiek:
		const pie = d3.pie().value((d) => d[1]);

		const data_ready = pie(Object.entries(data));

		// Maak de taartgrafiek: Elke deel van de taart is een pad dat we bouwen met behulp van de arc-functie.
		svgDonot
			.selectAll('whatever')
			.data(data_ready)
			.join('path')
			.attr(
				'd',
				d3
					.arc()
					.innerRadius(100) // Dit is de grootte van het gat in de donut
					.outerRadius(radius)
			)
			.attr('fill', (d) => color(d.data[0]))
			.attr('stroke', 'black')
			.style('stroke-width', '2px')
			.style('opacity', 0.7);
	});

    // Wisselt tussen meer of minder gegevens in de grafiek.
    function onClickShowMore() {
        // Soort ervoor dat de grafiek wisselt tussen de korte en lange versie van de grafiekdata.
		if (showLess) {
            showLess = false; // Schakelt naar de lange lijst.
            graphData = graphDataMore; // Gebruikt de lange lijst voor de grafiek.
        } else {
            showLess = true; // Schakelt terug naar de korte lijst.
            graphData = graphDataLess; // Gebruikt de korte lijst voor de grafiek.
        }
        // Verwijdert de oude grafiek en bouwt een nieuwe grafiek.
        document.getElementById('bar-chart').innerHTML = '';
        // Maakt de grafiek opnieuw met de nieuwe data.
        makeBarChart();
        // Toont de huidige status van de data in de console voor debuggen.
        console.log(graphDataMore, graphDataLess, graphData);
	}

</script>

<!-- HTML elementen voor de grafiek en de knop -->
<div id="bar-chart" />
<button on:click={() => onClickShowMore()}>Toon {showLess ? 'meer' : 'minder'}</button>

<!-- Toon de Metascore grafiek als een game deze heeft -->
{#if metaScore}
	<h3>Metascore: {metaScore}</h3>
{/if}
<div id="my_dataviz" />