<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    async function fetchApiData(url) {
        const response = await fetch(url);
        return await response.json();
    }

    onMount(async () => {
        // API URLs
        const dealsUrl = "https://www.cheapshark.com/api/1.0/deals?title=deus%20ex%20human%20revolution%20directors%20cut";
        const storesUrl = "https://www.cheapshark.com/api/1.0/stores";

        // Fetch API data
        const dealsData = await fetchApiData(dealsUrl);
        const storesData = await fetchApiData(storesUrl);
 
        let graphData = [];
        // Itereren door elk 'deal' object in de dealsData array
        for (let deal of dealsData) {
            // Zoekt naar de winkel in storesData die overeenkomt met de storeID van de huidige deal
            const store = storesData.find(s => s.storeID === deal.storeID);
            // Controleert of de overeenkomende winkel is gevonden
            if (store) {
                graphData.push({ storeName: store.storeName, price: parseFloat(deal.salePrice) }); // Voeg een object met winkelnaam en dealprijs aan de grafiek
                // parseFLoat is een string omzet in een floating-point getal (een getal met decimalen) om de price in een nieuw opject te zetten
            }
        }

        // Afmetingen en marges van de grafiek
        const margin = {top: 20, right: 30, bottom: 40, left: 90},
            width = 460 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;

        // SVG object toevoegen
        const svg = d3.select("#bar-chart")
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        // X axis (Horizontale As)
        const x = d3.scaleLinear()
            .domain([0, 100]) // Max prijs is 100 / moet nog aanpassen
            .range([0, width]);
        svg.append("g")
            .attr("transform", `translate(0, ${height})`)
            .call(d3.axisBottom(x));

        // Y axis (Verticale as)
        const y = d3.scaleBand()
            .range([0, height])
            .domain(graphData.map(d => d.storeName))
            .padding(.1);
        svg.append("g")
            .call(d3.axisLeft(y));

        // Staafdiagram toevoegen
        svg.selectAll("rect")
            .data(graphData)
            .join("rect")
            .attr("x", x(0))
            .attr("y", d => y(d.storeName))
            .attr("width", d => x(d.price))
            .attr("height", y.bandwidth())
            .attr("fill", "#69b3a2");

        // Tekstlabels toevoegen
        svg.selectAll(".label")
            .data(graphData)
            .enter()
            .append("text")
            .attr("class", "label")
            .attr("x", d => x(d.price) + 5) // ruimte van de prijs af
            .attr("y", d => y(d.storeName) + y.bandwidth() / 2 + 5) 
            .text(d => `€${d.price.toFixed(2)}`) // De Prijs 
            .attr("text-anchor", "start") // Tekst x-positie
            .style("fill", "black") // Kleur van de tekst
            .style("font-size", "12px"); // Grootte van de tekst
    });
</script>

<div id="bar-chart"></div>
