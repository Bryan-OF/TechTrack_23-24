<script>
	import Maincomponent from '../components/Maincomponent.svelte';

	const searchApi = 'https://www.cheapshark.com/api/1.0/games?title=';
	let searchInput = '';
	let searchedGameResults = [];
	let activeGame = null;

	// Functie om games te zoeken op basis van de zoekinput.
	async function fetchGameData() {
		const response = await fetch(searchApi + searchInput); // Haalt gegevens op van de API dat samenkomt met zoekinput.
		return await response.json(); // Zet de respons om naar JSON.
	}

	// Filtert de games zodat alleen de games worden weergeven die beginnen met de begin zoekinput.
	async function searchGame() {
		const games = await fetchGameData();

		// Filtert de games zodat alleen de games worden weergeven die beginnen met de begin zoekinput.
		searchedGameResults = games
			.filter((game) => game.external.toLowerCase().startsWith(searchInput.toLowerCase()))
			.slice(0, 10); // laat de eerste 10 resultaten zien.
	}

	function onClickGame(game) {
		activeGame = game; // Update actieve game.
		searchInput = ''; // Maakt Searchbar leeg.
		searchGame(); // Voert een nieuwe zoekopdracht uit.
	}
</script>

<main>
	<div class="search-container">
		<h1>Game<br />Checker</h1>
		<!-- Logo/ website -->
		<input
			type="text"
			id="myInput"
			bind:value={searchInput}
			on:input={searchGame}
			placeholder="Search for a Game"
		/>
		<!-- Zoekveld voor het invoeren van de game titel. -->
	</div>

	<div class="results-container">
		<ul>
			<!-- Toont zoekresultaten als knoppen. -->
			{#each searchedGameResults as game}
				<!-- Voor elk game wordt een knop gemaakt in de list -->
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
	:global(:root) {
		--border-radius: 8px;
		--box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
		--primary-bg-color: #fff;
		--secondary-bg-color: #f8f9fa;
		--text-color: #202124;
	}

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
		border-radius: 4px;
		margin-right: 15px;
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
		box-shadow: 0 1px 6px rgba(32, 33, 36, 0.28);
		font-size: 16px;
		color: #5f6368;
		outline: none;
	}

	#myInput:focus {
		box-shadow: 0 3px 8px rgba(32, 33, 36, 0.28);
	}

	.results-container {
		width: 100%;
		box-shadow: var(--box-shadow);
		border-radius: var(--border-radius);
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
		background-color: var(--primary-bg-color);
		padding: 12px 20px;
		border-bottom: 1px solid #e0e0e0;
		color: var(--text-color);
		font-size: 18px;
	}

	.game-item:hover {
		background-color: var(--secondary-bg-color);
		cursor: pointer;
	}
</style>
