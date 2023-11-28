<script>
	import Maincomponent from '../components/Maincomponent.svelte';

	const searchApi = 'https://www.cheapshark.com/api/1.0/games?title=';
	let searchInput;
	let searchedGameResults = [];
	let activeGame;

	async function searchGame() {
		const response = await fetch(searchApi + searchInput);
		const games = await response.json();
		searchedGameResults = games.slice(0, 10);
	}

	function onClickGame(game) {
		activeGame = null;
		setTimeout(() => {
			activeGame = game;
			searchInput = '';
			searchGame();
		}, 10);
	}
</script>

<main>
	<h1>Game Checker</h1>
	<input
		type="text"
		id="myInput"
		bind:value={searchInput}
		on:input={searchGame}
		placeholder="Search for a Game"
	/>

	<ul>
		{#each searchedGameResults as game}
			<button on:click={() => onClickGame(game)} class="game-item">
				<li>{game.external}</li>
				<img class="game-thumb" src={game.thumb} alt={game.external} />
			</button>
		{/each}
	</ul>
	{#if activeGame}
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
		border: 1px solid #ddd; /* Add a border to all links */
		margin-top: -1px; /* Prevent double borders */
		background-color: #f6f6f6; /* Grey background color */
		padding: 12px; /* Add some padding */
		text-decoration: none; /* Remove default text underline */
		font-size: 18px; /* Increase the font-size */
		color: black; /* Add a black text color */
		display: block; /* Make it into a block element to fill the whole list */
	}

	ul li button:hover:not(.header) {
		background-color: #eee; /* Add a hover effect to all links, except for headers */
	}

	.game-item {
		display: flex; /* Voeg flex display toe */
		align-items: center; /* Centreer de items verticaal */
		gap: 10px; /* Voeg wat ruimte toe tussen de naam en de thumbnail */
	}
</style>
