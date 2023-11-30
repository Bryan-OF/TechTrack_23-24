<script>
	import Maincomponent from '../components/Maincomponent.svelte';

	const searchApi = 'https://www.cheapshark.com/api/1.0/games?title=';
	let searchInput;
	let searchedGameResults = [];
	let activeGame;

    // Functie om games te zoeken op basis van de zoekinput.
	async function searchGame() {
		const response = await fetch(searchApi + searchInput); // Haalt gegevens op van de API dat samenkomt met zoekinput.
		const games = await response.json(); // Zet de respons om naar JSON.
		searchedGameResults = games.slice(0, 10); // Slaat de eerste 10 resultaten op.
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
		<h1>Game Checker</h1>
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
		<!-- Toont details van het actieve spel. -->
		<h3>{activeGame.external}</h3>
		<img src={activeGame.thumb} alt={activeGame.external} />
		<Maincomponent bind:activeGame />
	{/if}
</main>

<style>
	.game-thumb {
		max-width: 80px;
		max-height: 80px;
	}

	.search-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	#myInput {
		width: 700.8px;
		padding: 19.2px;
		border-radius: 30px;
		border: 2.4px solid #000;
		background: #fff;
		display: inline-flex;
		align-items: center;
		gap: 9.6px;
	}

	.results-container {
		height: 400px;
		overflow: auto;
	}

	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
		gap: 20px;
	}

	h1 {
		margin: 0;
		font-size: 400%;
	}

	ul {
		list-style-type: none;
		padding: 0;
		margin: 0;
		display: flex;
		flex-direction: column;
		align-items: flex-start;
	}

	ul li button {
		border: 1px solid #ddd;
		margin-top: -1px;
		background-color: #f6f6f6;
		padding: 12px;
		text-decoration: none;
		font-size: 18px;
		color: black;
		display: block;
	}

	ul li button:hover:not(.header) {
		background-color: #eee;
	}

	.game-item {
		display: flex;
		align-items: center;
		gap: 10px;
	}
</style>