<script context="module" lang="ts">
	export const prerender = true;
</script>

<script lang="ts">
	import Goal, { type GoalType } from '../lib/Goal.svelte';
	import Status from '../lib/Status.svelte';
	import { browser } from '$app/env';
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';
	import { flip } from 'svelte/animate';

	let lastSortSelection = 'mostRecentlyCreated';
	let existingGoals: GoalType[] = [];
	getExistingGoals();
	let selectedId = null;
	$: goalSelected = existingGoals.find((g) => g.id === selectedId);

	function getExistingGoals() {
		existingGoals =
			browser && localStorage.getItem('goals') ? JSON.parse(localStorage.getItem('goals')) : [];
		handleSortSelection(lastSortSelection);
	}

	const [send, receive] = crossfade({
		duration: (d) => Math.sqrt(d * 200),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 500,
				easing: quintOut,
				css: (t) => `
          transform: ${transform} scale(${t});
          opacity: ${t}
        `
			};
		}
	});

	const sortTypes = [
		{ label: 'Recently Created', value: 'mostRecentlyCreated' },
		{ label: 'Closest Target Date', value: 'targetDate' },
		{ label: 'Percentage Completed', value: 'percentageCompleted' },
		{ label: 'Urgent', value: 'urgent' }
	];

	function handleSortSelection(value: string) {
		lastSortSelection = value;
		const sortFunctions = {
			mostRecentlyCreated: () => {
				existingGoals = existingGoals.slice().sort((a, b) => (a.id < b.id ? -1 : 1));
			},
			targetDate: () => {
				existingGoals = existingGoals
					.slice()
					.sort((a, b) => (a.targetDate > b.targetDate ? -1 : 1));
			},
			percentageCompleted: () => {
				existingGoals = existingGoals.slice().sort((a, b) => {
					const aPercent = a.hoursComplete / a.estimatedHours;
					const bPercent = b.hoursComplete / b.estimatedHours;
					return aPercent + +a.done < bPercent + +b.done ? -1 : 1;
				});
			},
			urgent: () => {
				existingGoals = existingGoals.slice().sort((a, b) => {
					console.log(a.urgent, b.urgent, a.urgent > b.urgent);
					return +(a.urgent || 0) < +(b.urgent || 0) ? -1 : 1;
				});
			}
		};
		if (sortFunctions[value]) {
			sortFunctions[value]();
		}
	}
</script>

<svelte:head>
	<title>Goal Keeper</title>
</svelte:head>

<div class="h-full xl:w-4/6 m-auto">
	<section class="w-full xl:flex xl:justify-center">
		<!-- main screen -->
		<div class="flex items-center">
			<div class="w-full">
				{#if goalSelected}
					<div>
						<Goal bind:goal={goalSelected} on:goalAdded={getExistingGoals} />
					</div>
					<button
						class="btn mt-12 w-full"
						on:click={() => {
							selectedId = null;
						}}
					>
						Add Another Goal
					</button>
				{:else}
					<div>
						<Goal on:goalAdded={getExistingGoals} />
					</div>
				{/if}
			</div>
		</div>
	</section>

	<section class="xl:w-5/6 xl:flex m-auto justify-center">
		<span class="px-2">
			<Status goal={goalSelected} type="currentStatus" />
		</span>
		<span class="px-2">
			<Status goal={goalSelected} type="timeTilTarget" />
		</span>
		<span class="px-2">
			<Status goal={goalSelected} type="motivation" />
		</span>
	</section>
</div>

<!-- existing goals -->
<div
	class="px-4 xl:w-1.5/6 xl:scrollable h-full xl:fixed xl:mx-0 top-0 right-0 py-16 bg-opacity-5 bg-zinc-600"
>
	<div class="text-center font-bold">Goals</div>
	{#if !existingGoals?.length}
		Your goals will show up here as you add them
	{:else}
		<div class="text-center">
			<div>Sort By...</div>
			<select name="" on:change={(e) => handleSortSelection(e.target['value'])}>
				{#each sortTypes as { value, label }}
					<option {value}>{label}</option>
				{/each}
			</select>
		</div>
		<div class="scrollable flex pt-4 pb-12 xl:overflow-hidden xl:block">
			{#each existingGoals.slice().reverse() as goal (goal.id)}
				<div
					class="mb-3 cursor-pointer flex items-center justify-center hover:hover-state px-4"
					on:click={() => {
						selectedId = goal.id + '';
						setTimeout(() => {
							selectedId = goal.id;
						});
						window.scrollTo({
							top: 0,
							left: 0
						});
					}}
					in:receive={{ key: goal.id }}
					out:send={{ key: goal.id }}
					animate:flip={{ duration: 450 }}
				>
					<Goal bind:goal isSummary={true} viewId={selectedId} />
				</div>
			{/each}
		</div>
	{/if}
</div>

<style>
	section {
		margin-top: 2rem;
	}
</style>
