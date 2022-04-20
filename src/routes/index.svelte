<script lang="ts">
	import Dashboard from '$lib/Dashboard.svelte';
	// import AuthInteraction from '$lib/AuthInteraction.svelte';

	let signedIn = true;
</script>

{#if signedIn}
	<Dashboard />
{:else}
	<!-- <AuthInteraction /> -->
{/if}
