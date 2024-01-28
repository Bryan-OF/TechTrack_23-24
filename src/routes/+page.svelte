<script>
	import Maincomponent from '../components/Maincomponent.svelte';

	const searchApi = 'https://www.cheapshark.com/api/1.0/games?title=';
	let searchInput;
	let searchedGameResults = [];
	let activeGame;

    // Functie om games te zoeken op basis van de zoekinput.
	async function searchGame() {
		const response = await fetch(searchApi + searchInput); // Haalt gegevens op van de API dat samenkomt met zoekinput.
		let games = await response.json(); // Zet de respons om naar JSON.

		// Filtert de games zodat alleen de games worden weergeven die beginnen met de begin zoekinput.
		games = games.filter(game => game.external.toLowerCase().startsWith(searchInput.toLowerCase()));

		searchedGameResults = games.slice(0, 10); // laat de eerste 10 resultaten zien.
	}

    // Voort deze functies uit wanneer op een spel wordt geklikt.
	function onClickGame(game) {
		activeGame = null; // Reset de actieve game.
		setTimeout(() => { // Stelt uitvoering kort uit voor stabiliteit.
			activeGame = game; // Update actieve game.
			searchInput = ''; // Maakt Searchbar leeg.
			searchGame(); // Voert een nieuwe zoekopdracht uit.
		}, 10);
	}
</script>

<main>
	<div class="search-container">
		<h1>Game<br>Checker</h1>
		<!-- Zoekveld voor het invoeren van de game titel. -->
		<input
			type="text"
			id="myInput"
			bind:value={searchInput}
			on:input={searchGame}
			placeholder="Search for a Game"
		/>
	</div>

	<div class="results-container">
		<ul>
			<!-- Toont zoekresultaten als knoppen. -->
			{#each searchedGameResults as game}
				<button on:click={() => onClickGame(game)} class="game-item">
					<img class="game-thumb" src={game.thumb} alt={game.external} />
					<li>{game.external}</li>
				</button>
			{/each}
		</ul>
	</div>

	{#if activeGame}
		<Maincomponent bind:activeGame />
	{/if}
</main>

<style>
	main {
		width: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		gap: 20px;
	}

	.game-thumb {
    	max-width: 100px;
    	max-height: 150px;
    	min-width: 50px;
    	min-height: 50px;

    	border-radius: 4px; /* Licht afgeronde hoeken voor afbeeldingen */
    	margin-right: 15px; /* Ruimte tussen afbeelding en tekst */
	}

	.search-container {
    	display: flex;
    	flex-direction: row;
    	align-items: center;
    	justify-content: center;
    	gap: 50px;
    	width: 100%;
    	margin-top: 20px;
	}

	#myInput {
    	width: 100%;
    	padding: 12px 24px;
    	border-radius: 24px;
    	border: 1px solid #dfe1e5;
    	background: #fff;
    	display: inline-flex;
    	align-items: center;
    	gap: 9.6px;
    	flex-grow: 1;
    	box-shadow: 0 1px 6px rgba(32,33,36,0.28);
    	font-size: 16px;
    	color: #5F6368;
    	outline: none;
	}

	#myInput:focus {
    	box-shadow: 0 3px 8px rgba(32,33,36,0.28);
	}

	.results-container {
    	width: 100%;
    	box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    	border-radius: 8px;
    	overflow: hidden;
	}

	h1 {
		margin: 0;
		font-size: 180%;
	}

	ul {
    	list-style-type: none;
    	padding: 0;
    	margin: 0;
	}

	button {
        border: none;
    }

	ul li button:hover:not(.header) {
		background-color: #eee;
	}

    .game-item {
        display: flex;
        flex-direction: row;
        align-items: center;
        width: 100%;
        background-color: #fff;
        padding: 12px 20px;
        border-bottom: 1px solid #e0e0e0;
        color: #202124;
        font-size: 18px;
    }

	.game-item:hover {
		background-color: #f8f9fa;
		cursor: pointer;
	}
</style>




