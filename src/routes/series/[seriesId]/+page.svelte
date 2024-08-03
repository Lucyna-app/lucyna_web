<script>
	import { onMount } from 'svelte';
	import axios from 'axios';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	let characters = [];
	let seriesId;
	let seriesName = '';

	onMount(async () => {
		seriesId = $page.params.seriesId;
		await fetchSeriesData();
	});

	async function fetchSeriesData() {
		try {
			const response = await axios.get(`http://localhost:8000/series/${seriesId}/characters`);
			characters = response.data.characters;
			seriesName = response.data.series_name; // Assuming the API returns the series name
		} catch (error) {
			console.error('Error fetching series data:', error);
		}
	}

	async function createCharacter(event) {
		// ... (keep existing code)
	}

	function navigateToAllSeries() {
		goto('/');
	}
</script>

<button on:click={navigateToAllSeries} style="margin-bottom: 20px;">Back to All Series</button>

<h1>{seriesName}</h1>

<h2>Characters in Series</h2>
<ul>
	{#each characters as character}
		<li><a href="/series/{seriesId}/character/{character[0]}">{character[1]}</a></li>
	{/each}
</ul>

<h2>Create New Character</h2>
<form on:submit={createCharacter}>
	<input name="character_name" placeholder="Character Name" required />
	<input name="series_uuid4" value={seriesId} type="hidden" />
	<input name="rarity" type="number" min="1" max="5" required />
	<button type="submit">Create Character</button>
</form>
