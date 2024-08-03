<script>
	import { onMount } from 'svelte';
	import axios from 'axios';

	let series = [];

	onMount(async () => {
		const response = await axios.get('http://localhost:8000/series/read_all');
		series = response.data.all_series;
	});
</script>

<h1>Game Administrator</h1>
<h2>Series</h2>
<ul>
	{#each series as s}
		<li><a href="/series/{s[0]}">{s[1]}</a></li>
	{/each}
</ul>
