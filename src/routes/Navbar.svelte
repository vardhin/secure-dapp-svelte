<script>
	import { goto } from '$app/navigation';
	import { darkMode } from '../lib/utils/stores';
	import { get } from 'svelte/store';
	export let title = ''; // Ensure title is passed as a prop

	let isDarkMode = false;
	darkMode.subscribe((value) => {
		isDarkMode = value;
	});

	const goBack = () => {
		history.back();
	};

	const toggleTheme = () => {
		darkMode.set(!get(darkMode));
	};
</script>

<nav class:dark-mode={isDarkMode}>
	<div class="back-button" on:click={goBack}>
		<span>&larr;</span>
	</div>
	<div class="title">
		<h1>{title}</h1>
		<!-- Ensure title is reactive and passed -->
	</div>

	<div class="theme-toggle" on:click={toggleTheme}>
		{#if isDarkMode}
			<span>☀️</span>
		{:else}
			<span>🌙</span>
		{/if}
	</div>
</nav>

<style>
	nav {
		display: flex;
		align-items: center;
		justify-content: space-between;
		background-color: var(--navbar-bg-color);
		color: var(--navbar-text-color);
		padding: 10px 20px;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
		transition:
			background-color 0.3s ease,
			color 0.3s ease;
	}

	.dark-mode {
		--navbar-bg-color: #222;
		--navbar-text-color: #ddd;
	}

	.back-button {
		cursor: pointer;
		font-size: 1.2rem;
	}

	.title h1 {
		font-size: 1.5rem;
		margin: 0;
	}

	.theme-toggle {
		cursor: pointer;
		font-size: 1.5rem;
	}
</style>
