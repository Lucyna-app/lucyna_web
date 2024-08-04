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
	let fileInput;
	let previewImage = null;
	let updateArtUuid = null;
	const DISPLAY_WIDTH = 265;
	const DISPLAY_HEIGHT = 371;

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

	function handleFileSelect(event) {
		const file = event.target.files[0];
		if (file) {
			const reader = new FileReader();
			reader.onload = (e) => {
				const img = new Image();
				img.onload = () => {
					if (img.width === 375 && img.height === 525) {
						previewImage = e.target.result;
					} else {
						alert('Image must be 375x525 pixels');
						fileInput.value = '';
					}
				};
				img.src = e.target.result;
			};
			reader.readAsDataURL(file);
		}
	}

	async function updateArt(artUuid) {
		if (!fileInput.files[0]) {
			alert('Please select a file to update');
			return;
		}

		if (confirm('Are you sure you want to update this art?')) {
			const formData = new FormData();
			formData.append('art', fileInput.files[0]);

			try {
				await axios.put(`http://localhost:8000/art/update/${artUuid}`, formData, {
					headers: { 'Content-Type': 'multipart/form-data' }
				});
				alert('Art updated successfully');
				await fetchCharacterData();
				previewImage = null;
				fileInput.value = '';
				updateArtUuid = null;
			} catch (error) {
				console.error('Error updating art:', error);
				alert('Error updating art. Please try again.');
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
	<div class="art-container">
		{#each artUrls as url, index}
			<div class="art-item">
				<img src={url} alt="Character Art" class="art-image" on:click={() => openLargeImage(url)} />
				<button
					class="update-button"
					on:click={() => (updateArtUuid = url.split('/').pop().split('?')[0])}
				>
					Update
				</button>
			</div>
		{/each}
	</div>
{:else}
	<p>No arts available for this character.</p>
{/if}

{#if updateArtUuid}
	<div>
		<h3>Update Art</h3>
		<input type="file" accept="image/*" bind:this={fileInput} on:change={handleFileSelect} />
		{#if previewImage}
			<img
				src={previewImage}
				alt="Preview"
				style="width: {DISPLAY_WIDTH}px; height: {DISPLAY_HEIGHT}px; object-fit: cover;"
			/>
		{/if}
		<button on:click={() => updateArt(updateArtUuid)}>Confirm Update</button>
		<button
			on:click={() => {
				updateArtUuid = null;
				previewImage = null;
				fileInput.value = '';
			}}>Cancel</button
		>
	</div>
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

<style>
	.art-container {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(265px, 1fr));
		gap: 20px;
	}

	.art-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.art-image {
		width: 265px;
		height: 371px;
		object-fit: cover;
		cursor: pointer;
	}

	.update-button {
		margin-top: 10px;
	}
</style>
