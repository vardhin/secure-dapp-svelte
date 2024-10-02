<script>
	import { onMount } from 'svelte';
	import { getAllContacts } from '../lib/utils/contactManager';
	import { goto } from '$app/navigation'; // SvelteKit's navigation helper
	import Navbar from './Navbar.svelte';
	import { darkMode } from '../lib/utils/stores';

	let contacts = [];
	let visible = false;
	let isDarkMode;

	// Fetch all contacts when the component mounts or when a new contact is added
	const fetchContacts = async () => {
		const availableContacts = await getAllContacts();
		contacts = availableContacts;
	};

	// This works like useEffect in React
	onMount(() => {
		fetchContacts();
		setTimeout(() => {
			visible = true; // Trigger transitions on mount
		}, 100);
	});

	// Function to handle contact button click and route to ChatWindow
	const handleContactClick = (contactName) => {
		goto('/chat', { state: { contactName } });
	};

	// Function to navigate to create contact page
	const navigateToCreateContact = () => {
		goto('/contact');
	};

	darkMode.subscribe((value) => {
		isDarkMode = value;
	});
</script>

<body>
	<Navbar title="Recon" />
	<main class:dark-mode={isDarkMode}>
		<ul>
			{#each contacts as contact}
				<li>
					<button on:click={() => handleContactClick(contact)}>
						{contact}
					</button>
				</li>
			{/each}
		</ul>

		<!-- Floating "+" button to navigate to create contact page -->
		<button class="floating-button" on:click={navigateToCreateContact}> + </button>
	</main>
</body>

<style>
	@import '@fontsource/orbitron/400.css';
	@import '@fontsource/orbitron/700.css';

	:root {
		--bg-color: #e7dfe8;
		--form-bg-color: #fff;
		--text-color: #7ddf64;
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
    --text-color: #a6e22e;
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
		flex-direction: column;
		align-items: center;
		height: 100vh;
		padding: 20px;
		transition:
			background-color 0.3s ease,
			color 0.3s ease;
		position: relative;
	}

	body {
		margin: 0; /* Remove default margin */
		padding: 0; /* Remove default padding */
		box-sizing: border-box; /* Ensure consistent box sizing */
		height: 100%; /* Make sure body takes full height */
		width: 100%; /* Make sure body takes full width */
		font-family: 'Orbitron', sans-serif;
	}

	h1 {
		color: var(--text-color);
		font-size: 2rem;
		margin-bottom: 20px;
		transition: color 0.3s ease;
	}

	ul {
		list-style-type: none;
		padding: 0;
		margin: 0;
		width: 100%;
		max-width: 400px;
	}

	li {
		margin-bottom: 10px;
		width: 100%;
		text-align: center;
	}

	button {
		background-color: var(--button-bg-color);
		color: #fff;
		padding: 10px;
		border: none;
		border-radius: 8px;
		width: 100%;
		max-width: 400px;
		font-size: 1.1rem;
		letter-spacing: 2px;
		cursor: pointer;
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

	/* Floating "+" button */
	.floating-button {
		position: fixed;
		bottom: 30px;
		right: 30px;
		width: 60px;
		height: 60px;
		border-radius: 50%;
		background-color: var(--button-bg-color);
		color: white;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 2rem;
		border: none;
		cursor: pointer;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
		transition:
			background-color 0.3s ease,
			transform 0.2s ease,
			box-shadow 0.2s ease;
	}

	.floating-button:hover {
		background-color: var(--button-hover-bg-color);
		transform: translateY(-5px);
		box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
	}

	.floating-button:active {
		transform: translateY(2px);
	}
</style>
