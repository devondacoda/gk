<script lang="ts">
	import Header from '$lib/header/Header.svelte';
	import '../app.css';
</script>

<Header />

<main>
	<slot />
</main>

<style>
	main {
		flex: 1;
		display: flex;
		flex-direction: column;
		padding: 3rem 1rem 2rem;
		width: 100%;
		margin: 0;
		box-sizing: border-box;
	}
</style>
