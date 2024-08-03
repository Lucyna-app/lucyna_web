<script>
	import { onMount } from 'svelte';
	import axios from 'axios';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	let character = null;
	let artUrls = [];
	let seriesId;
	let characterId;
	let selectedImage = null;

	onMount(async () => {
		seriesId = $page.params.seriesId;
		characterId = $page.params.characterId;
		await fetchCharacterData();
	});

	async function fetchCharacterData() {
		try {
			const response = await axios.get(
				`http://localhost:8000/series/${seriesId}/character/${characterId}`
			);
			character = response.data.character;
			artUrls = response.data.art_urls || [];
			console.log('Fetched data:', response.data); // Add this line for debugging
		} catch (error) {
			console.error('Error fetching character data:', error);
			artUrls = [];
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
				'Are you sure you want to delete this character? This will also delete all associated art and cards. This action cannot be undone.'
			)
		) {
			try {
				await axios.delete(`http://localhost:8000/series/${seriesId}/character/${characterId}`);
				alert('Character, associated art, and cards deleted successfully');
				goto(`/series/${seriesId}`);
			} catch (error) {
				console.error('Error deleting character:', error);
				alert('Error deleting character. Please try again.');
			}
		}
	}

	function openLargeImage(url) {
		selectedImage = url;
	}

	function closeLargeImage() {
		selectedImage = null;
	}
</script>

<button on:click={() => goto(`/series/${seriesId}`)} style="margin-bottom: 20px;"
	>Back to Series</button
>

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
{#if artUrls && artUrls.length > 0}
	<div
		style="display: grid; grid-template-columns: repeat(auto-fill, minmax(375px, 1fr)); gap: 20px;"
	>
		{#each artUrls as url}
			<img
				src={url}
				alt="Character Art"
				style="width: 375px; height: 525px; object-fit: cover; cursor: pointer;"
				on:click={() => openLargeImage(url)}
			/>
		{/each}
	</div>
{:else}
	<p>No arts available for this character.</p>
{/if}

{#if selectedImage}
	<div
		style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); display: flex; justify-content: center; align-items: center; z-index: 1000;"
		on:click={closeLargeImage}
	>
		<img
			src={selectedImage}
			alt="Large Character Art"
			style="max-width: 90%; max-height: 90%; object-fit: contain;"
		/>
	</div>
{/if}

<h2>Upload New Art</h2>
<form on:submit={uploadArt}>
	<input type="file" name="art" accept="image/*" required />
	<button type="submit">Upload Art</button>
</form>
