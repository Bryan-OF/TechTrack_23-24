<script>
	import Chart from '../components/BarCharttest1.svelte';
	import Barcharttest2 from '../components/Barcharttest2.svelte';
	import Linechart from '../components/Linecharttest1.svelte';
	import MonumentsList from '../components/MonumentsList.svelte';
	import Search from '../components/Search.svelte';

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
		activeGame = game;
		searchInput = '';
		searchGame();
	}
</script>

<main>
	<input
		type="text"
		id="myInput"
		bind:value={searchInput}
		on:keydown={searchGame}
		placeholder="Search for names.."
	/>

	<ul>
		{#each searchedGameResults as game}
			<div on:click={onClickGame(game)}>
				<li>{game.external}</li>
				<img src={game.thumb} alt={game.external} />
			</div>
		{/each}
	</ul>
	{#if activeGame}
		<h3>{activeGame.external}</h3>
		<img src={activeGame.thumb} alt={activeGame.external} />
        <Barcharttest2 bind:activeGame />
	{/if}
</main>

<style>
	ul {
		/* Remove default list styling */
		list-style-type: none;
		padding: 0;
		margin: 0;
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
</style>
