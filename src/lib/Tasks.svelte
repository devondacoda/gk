<script context="module" lang="ts">
	export type TaskType = {
		id: number;
		description: string;
		done: boolean;
	};
</script>

<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import type { GoalType } from './Goal.svelte';
	export let goal: GoalType = null;

	const emptyTaskState: TaskType = {
		id: -1,
		description: '',
		done: false
	};
	const dispatch = createEventDispatcher();

	let newTask = { ...emptyTaskState };

	function insertTask() {
		if (!goal) return;
		if (!goal.tasks) goal.tasks = [];
		goal.tasks.push({ ...newTask, id: goal.tasks.length + 1 });
		goal = goal;
		newTask = { ...emptyTaskState };
		dispatch('taskAdded', newTask);
	}
</script>

<div class="card px-6 bg-emerald-600 bg-opacity-80">
	<div class="text-center font-bold">Tasks</div>
	<div class="text-center py-4">
		<form on:submit|preventDefault={insertTask} class="app-input">
			<input
				id="task-description"
				type="text"
				autocomplete="off"
				placeholder="Enter Task for Goal"
        required
				bind:value={newTask.description}
			/>
      <button class="btn">Add Task</button>
		</form>
	</div>

	{#if goal && goal.tasks}
    <div class="scrollable tasks-list -mx-6">
      <div class="px-6">
        {#each goal.tasks.slice().reverse() || [] as task (goal.id + '.' + task.id)}
          <div class="card mb-8 shadow-md" class:done={task.done}>
            <div class="card-header">
              <div class="mr-2">#{goal.id}.{task.id}</div>
            </div>
            <div class="card-body">
              <div class="flex items-center">
                <span>
                  <button
                    class="circle toggle-done mr-2 app-border"
                    aria-label="Mark as done"
                    on:click={() => {
                      task.done = !task.done;
                      dispatch('taskToggled', task);
                    }}
                  />
                </span>
                <p>{task.description}</p>
              </div>
            </div>
          </div>
        {/each}
      </div>
    </div>
	{/if}
</div>

<style>
  .tasks-list {
    max-height: calc(100vh - 480px);
  }
</style>
