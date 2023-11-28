<!-- Eerst zou ik meerdere componenten oproepen, maar nu werk ik met 1 maincomponent. Source: https://www.w3schools.com/howto/howto_js_filter_lists.asp -->
<script>
    const searchApi = "https://www.cheapshark.com/api/1.0/games?title="
    let searchInput;
    let searchedGameResults = [];

    async function searchGame() {
        const response = await fetch(searchApi+searchInput);
        const games = await response.json();
        searchedGameResults = games.slice(0, 10);
    }
</script>

<input type="text" id="myInput" bind:value={searchInput} on:keydown={searchGame} placeholder="Search for names..">

<ul>
	{#each searchedGameResults as game}
		<li>{game.external}</li>
        <img src={game.thumb} alt="">
	{/each}
</ul>

<style>
    #myInput {
        background-image: url('/css/searchicon.png'); /* Add a search icon to input */
        background-position: 10px 12px; /* Position the search icon */
        background-repeat: no-repeat; /* Do not repeat the icon image */
        width: 100%; /* Full-width */
        font-size: 16px; /* Increase font-size */
        padding: 12px 20px 12px 40px; /* Add some padding */
        border: 1px solid #ddd; /* Add a grey border */
        margin-bottom: 12px; /* Add some space below the input */
    }

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