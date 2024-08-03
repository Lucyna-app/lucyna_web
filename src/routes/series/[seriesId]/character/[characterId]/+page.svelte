<script>
	import { onMount } from 'svelte';
	import axios from 'axios';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	let character = null;
	let arts = [];
	let seriesId;
	let characterId;

	onMount(async () => {
		seriesId = $page.params.seriesId;
		characterId = $page.params.characterId;
		await fetchCharacterData();
	});

	function navigateToSeries() {
		goto(`/series/${seriesId}`);
	}

	async function fetchCharacterData() {
		try {
			const response = await axios.get(
				`http://localhost:8000/series/${seriesId}/character/${characterId}`
			);
			character = response.data.character;
			arts = response.data.arts || [];
		} catch (error) {
			console.error('Error fetching character data:', error);
		}
	}

	async function updateCharacter(event) {
		event.preventDefault();
		const formData = new FormData(event.target);
		const characterData = Object.fromEntries(formData);
		try {
			await axios.put(
				`http://localhost:8000/series/${seriesId}/character/${characterId}`,
				characterData
			);
			await fetchCharacterData();
		} catch (error) {
			console.error('Error updating character:', error);
		}
	}

	async function uploadArt(event) {
		event.preventDefault();
		const formData = new FormData(event.target);
		formData.append('character_uuid4', characterId);
		try {
			await axios.post('http://localhost:8000/art/create', formData, {
				headers: { 'Content-Type': 'multipart/form-data' }
			});
			await fetchCharacterData();
		} catch (error) {
			console.error('Error uploading art:', error);
		}
	}

	async function deleteCharacter() {
		if (
			confirm(
				'Are you sure you want to delete this character and all associated art? This action cannot be undone.'
			)
		) {
			try {
				await axios.delete(`http://localhost:8000/series/${seriesId}/character/${characterId}`);
				alert('Character and associated art deleted successfully');
				goto(`/series/${seriesId}`);
			} catch (error) {
				console.error('Error deleting character:', error);
				alert('Error deleting character. Please try again.');
			}
		}
	}
</script>

<button on:click={navigateToSeries} style="margin-bottom: 20px;">Back to Series</button>

<h1>{character ? character[1] : 'Loading...'}</h1>

{#if character}
	<form on:submit={updateCharacter}>
		<input name="name" value={character[1]} />
		<input name="rarity" type="number" value={character[3]} min="1" max="5" />
		<button type="submit">Update Character</button>
	</form>

	<button on:click={deleteCharacter} style="background-color: red; color: white; margin-top: 20px;"
		>Delete Character</button
	>
{/if}

<h2>Arts</h2>
{#if arts.length > 0}
	<ul>
		{#each arts as art}
			<li>{art[0]}</li>
		{/each}
	</ul>
{:else}
	<p>No arts available for this character.</p>
{/if}

<h2>Upload New Art</h2>
<form on:submit={uploadArt}>
	<input type="file" name="art" accept="image/*" required />
	<button type="submit">Upload Art</button>
</form>
