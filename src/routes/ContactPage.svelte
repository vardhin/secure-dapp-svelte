<script>
	import { createContact, saveProfilePicture } from '../lib/utils/contactManager';
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';
	import { generateRandomString, generateQRCode } from '../lib/utils/qrUtils.js';
	import Navbar from './Navbar.svelte'; // Assuming Navbar is in the same directory
	import { darkMode } from '../lib/utils/stores'; // Import dark mode store

	let name = '';
	let id = '';
	let friendId = '';
	let profilePicture = null;
	let visible = false; // Track form visibility to trigger animations
	let qrCodeDataUrl = '';
	let isDarkMode = false;

	// Create a new contact with profile picture
	const handleCreateContact = async () => {
		if (!profilePicture) {
			alert('Please upload a profile picture!');
			return;
		}

		// Convert the profile picture to base64
		const reader = new FileReader();
		reader.onloadend = async () => {
			const base64String = reader.result.replace('data:', '').replace(/^.+,/, '');

			// First, create the contact
			await createContact(name, id, friendId);

			// Then, save the profile picture for the contact
			await saveProfilePicture(name, base64String);
			alert(`Contact ${name} created successfully with a profile picture!`);
		};
		reader.readAsDataURL(profilePicture);
	};

	// Ensure the animation triggers on page load
	onMount(async () => {
		setTimeout(() => {
			visible = true; // Set to true after a short delay to trigger transitions
		}, 100); // Short delay before showing forms

		id = generateRandomString();
		qrCodeDataUrl = await generateQRCode(id);
	});

	// Subscribe to darkMode store
	darkMode.subscribe((value) => {
		isDarkMode = value;
	});
</script>

<!-- Orbitron font -->
<link
	href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap"
	rel="stylesheet"
/>

<body>
	<Navbar title="Create Contact" />
	<main class:dark-mode={isDarkMode}>
		<div class="App">
			{#if visible}
				<!-- Form to Create Contact with Profile Picture -->
				<form on:submit|preventDefault={handleCreateContact} in:fade out:fade class="form">
					<input type="text" bind:value={name} placeholder="Friend Name" required />
					<input type="text" bind:value={friendId} placeholder="Friend ID" />
					<label class="file-input">
						<input
							type="file"
							accept="image/*"
							on:change={(e) => (profilePicture = e.target.files[0])}
							required
						/>
						<span>{profilePicture ? profilePicture.name : 'Friend Profile Picture'}</span>
					</label>
					<div class="qr-container">
						<p>Ask your friend to copy/scan this ID</p>
						<p>Your ID for Friend:</p>
						<p>{id}</p>
						{#if qrCodeDataUrl}
							<img src={qrCodeDataUrl} alt="QR Code" />
						{/if}
					</div>
					<p></p>

					<button type="submit">Create Contact</button>
				</form>
			{/if}
		</div>
	</main>
</body>

<style>
	@import '@fontsource/orbitron/400.css'; /* For regular weight */
	@import '@fontsource/orbitron/700.css'; /* For bold weight */

	:root {
		--bg-color: #e7dfe8;
		--form-bg-color: #fff;
		--text-color: #36827f;
		--button-bg-color: #0c7489;
		--button-hover-bg-color: #13505b;
		--body-bg-color: #e7dfe8;
		--body-text-color: #000;
		--navbar-bg-color: #f5f5f5;
		--navbar-text-color: #333;
	}

	.dark-mode {
    --bg-color: #121212;
    --form-bg-color: #1e1e1e;
    --text-color: #e0e0e0;
    --button-bg-color: #36827f; /* Light green for button background */
    --button-hover-bg-color: #2a6765; /* Dark teal for hover state */
    --body-bg-color: #00171f;
    --body-text-color: #ffffff; /* Change to white for better readability */
    --navbar-bg-color: #2a353b;
    --navbar-text-color: #e0e0e0; /* Light gray for navbar text */
}

	main {
		background-color: var(--body-bg-color);
		color: var(--body-text-color);
		font-family: 'Orbitron', sans-serif;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100vh;
		width: 100%;
		transition:
			background-color 0.3s ease,
			color 0.3s ease;
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	body {
		font-family: 'Orbitron', sans-serif;
		margin: 0; /* Remove default margin */
		padding: 0; /* Remove default padding */
		box-sizing: border-box; /* Ensure consistent box sizing */
		height: 100%; /* Make sure body takes full height */
		width: 100%; /* Make sure body takes full width */
	}

	.qr-container {
		text-align: center;
		margin-top: 20px;
	}

	img {
		margin-top: 10px;
	}

	p {
		font-family: monospace;
		font-size: 16px;
		word-break: break-all;
	}

	input,
	button,
	.file-input span {
		font-family: 'Orbitron', sans-serif;
	}

	.App {
		padding: 5px;
		min-height: 80vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	form {
		background: var(--form-bg-color);
		border-radius: 12px;
		box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
		padding: 25px;
		width: 100%;
		max-width: 350px;
		text-align: center;
		transition:
			transform 0.3s ease,
			box-shadow 0.3s ease;
	}

	form:hover {
		transform: translateY(-5px);
		box-shadow: 0 12px 24px rgba(0, 0, 0, 0.4);
	}

	h2 {
		color: var(--text-color);
		font-size: 1.8rem;
		margin-bottom: 20px;
	}

	input {
		display: block;
		width: 93%;
		padding: 12px;
		margin-bottom: 15px;
		border: 1px solid var(--input-border-color);
		border-radius: 8px;
		background-color: var(--input-bg-color);
		color: var(--text-color);
		font-size: 1rem;
		outline: none;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) inset;
		transition:
			background-color 0.3s ease,
			box-shadow 0.3s ease;
	}

	input:focus {
		background-color: var(--input-bg-color);
		box-shadow: 0 0 5px var(--input-border-color);
	}

	.file-input {
		display: block;
		position: relative;
		cursor: pointer;
		margin-bottom: 13px;
	}

	.file-input input[type='file'] {
		opacity: 0;
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		cursor: pointer;
	}

	.file-input span {
		display: block;
		padding: 12px;
		background-color: var(--input-bg-color);
		border: 2px dashed var(--input-border-color);
		border-radius: 8px;
		color: var(--text-color);
		transition: background-color 0.3s ease;
	}

	.file-input:hover span {
		background-color: var(--input-bg-color);
	}

	button {
		background-color: var(--button-bg-color);
		color: #fff;
		padding: 12px;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		width: 100%;
		font-size: 1.1rem;
		letter-spacing: 2px;
		transition:
			background-color 0.3s ease,
			transform 0.2s ease;
	}

	button:hover {
		transform: translateY(-2px);
		background-color: var(--button-hover-bg-color);
	}

	button:active {
		transform: translateY(1px);
	}

	.theme-toggle {
		margin-top: 20px;
		background-color: var(--button-bg-color);
		color: white;
		padding: 10px 20px;
		border-radius: 6px;
		border: none;
		cursor: pointer;
		transition: background-color 0.3s ease;
	}

	.theme-toggle:hover {
		background-color: var(--button-hover-bg-color);
	}

	@media (max-width: 600px) {
		form {
			width: 90%;
			padding: 15px;
			max-width: 300px;
		}

		h2 {
			font-size: 1.5rem;
		}

		input {
			padding: 10px;
			font-size: 0.9rem;
		}

		button {
			font-size: 1rem;
			padding: 10px;
		}
	}

	@media (min-width: 601px) {
		form {
			width: 100%;
			max-width: 400px;
		}

		h2 {
			font-size: 2rem;
		}

		input {
			font-size: 1.1rem;
		}

		button {
			font-size: 1.2rem;
			padding: 14px;
		}
	}
</style>
