<script lang="ts" context="module">
	export type GoalASDe = {
		id: number;
		uid: string;
		createdAt: Date;
		objective: string;
		done: boolean;
		targetDate: Date | string;
		motivation: string;
		estimatedHours: number;
		hide?: boolean;
		hoursComplete?: number;
	};
</script>

<script lang="ts">
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

  export let goal = null;

	const progress = tweened(0, {
		duration: 400,
		easing: cubicOut
	});
</script>

<progress value={$progress}></progress>

<button on:click="{() => progress.set(+((goal?.hoursComplete || 0) / (goal?.estimatedHours || 0)).toFixed(2))}">
	Dynamic%
</button>

<button on:click="{() => progress.set(0.25)}">
	25%
</button>

<button on:click="{() => progress.set(0.5)}">
	50%
</button>

<button on:click="{() => progress.set(0.75)}">
	75%
</button>

<button on:click="{() => progress.set(1)}">
	100%
</button>

<style>
	progress {
		display: block;
		width: 100%;
	}
</style>