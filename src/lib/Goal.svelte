<script lang="ts" context="module">
  export type GoalType = {
    id: number;
    uid: string;
    createdAt: Date;
    objective: string;
    done: boolean;
    targetDate: string;
    motivation: string;
    estimatedHours: number;
    hide?: boolean;
    hoursComplete?: number;
    urgent?: boolean;
    tasks: TaskType[];
  };
</script>

<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { onMount } from 'svelte';
  // import Note from '../lib/Note.svelte';
  import { tweened } from 'svelte/motion';
  import { cubicOut, quintOut } from 'svelte/easing';
import type { TaskType } from './Tasks.svelte';
import Tasks from './Tasks.svelte';
import { crossfade } from 'svelte/transition';

  export let goal: GoalType = null;
  export let isSummary: boolean = false;
  export let viewId: number = -1;

  let today: Date = new Date();
  today.setUTCHours(0, 0 , 0, 0);

  const dispatch = createEventDispatcher();
  const emptyState: GoalType = {
    objective: '',
    targetDate: '',
    motivation: '',
    done: false,
    estimatedHours: null,
    createdAt: today,
    uid: 'test_1',
    id: -1,
    hoursComplete: 0,
    urgent: false,
    tasks: []
  };

  let newGoal: GoalType = { ...emptyState };

  let confetti = null;
  onMount(() => {
    confetti = window['confetti'];
  });

  const progress = tweened(0, {
    duration: 800,
    easing: cubicOut
  });

  $: {
    progress.set(+((goal?.hoursComplete || 0) / (goal?.estimatedHours || 0)).toFixed(2))
  }

  function getDateAsString(date: Date) {
    return date.toUTCString().split(':')[0].slice(0, -2);
  }

  function saveGoal(goalToAdd: GoalType) {
    const existingGoals = localStorage.getItem('goals');
    let arr: GoalType[] = existingGoals ? JSON.parse(existingGoals) : [];
    const goalIdx = arr.findIndex((goal) => goal.id === goalToAdd.id);
    if (goalIdx > -1) {
      arr[goalIdx] = goalToAdd;
    } else {
      goalToAdd.id = arr.length + 1;
      arr = arr.concat(goalToAdd);
    }
    localStorage.setItem(`goals`, JSON.stringify(arr));
    dispatch('goalAdded', goalToAdd);
    newGoal = { ...emptyState };
  }

  function handleCreate() {
    saveGoal(newGoal);
  }

  let timer = null;
  function debounceSave() {
    clearTimeout(timer);
    timer = setTimeout(() => {
      if (goal) {
        if (goal.hoursComplete < 0) goal.hoursComplete = 0;
        saveGoal(goal);
      };
    }, 750);
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
</script>

<div class="xl:flex justify-center">
  {#if goal && !isSummary}
    <div class="px-4 xl:w-1/6 absolute left-0">
      <Tasks bind:goal on:taskAdded={() => saveGoal(goal)} on:taskToggled={() => saveGoal(goal)} />
    </div>
  {/if}

  <div
    in:receive={{ key: (goal || newGoal).id }}
    class="card min-w-min"
    class:urgent={newGoal?.urgent || goal?.urgent}
    class:done={goal?.done}
    class:app-border={viewId === goal?.id}
    class:shadow-none={viewId === goal?.id}
  >
    {#if goal}
      <!-- existing goal -->
      <div class="card-header max-w-sm truncate" class:w-40={isSummary}>
        <span class="float-left mr-2">#{goal.id}</span>
        {goal.objective}
        {#if goal?.urgent}
          <span class:circle={isSummary} class="urgent-tag bg-yellow-100 text-yellow-800 text-xs font-semibold mr-2 px-2.5 py-0.5 rounded">
            {isSummary ? '' : 'URGENT'}
          </span>
        {/if}
      </div>
      <div class="card-body">
        {#if isSummary}
          <p class="text-center">
            {Math.floor((goal.hoursComplete / goal.estimatedHours) * 100)}% Time Spent
          </p>
          <progress value={$progress}></progress>
        {:else}
          <form on:submit|preventDefault={() => saveGoal(goal)}>
            <label for="hoursComplete">Total Hours Logged</label>
            <input
              type="number"
              name="hoursComplete"
              autocomplete="off"
              min="0"
              max={goal.estimatedHours}
              placeholder="Enter Hours Complete"
              bind:value={goal.hoursComplete}
              required
              on:input={() => debounceSave()}
            /> / {goal.estimatedHours}
          </form>
          <section class="goal-updates">
            <div class="flex items-center">
              <span>
                <button
                  class="circle toggle-done mr-2 app-border"
                  aria-label="Mark as done"
                  on:click={() => {
                    goal.done = !goal.done;
                    goal = goal;
                    saveGoal(goal);
                    if (goal.done && confetti) {
                      confetti({
                        origin: {
                          x: 0.45
                        }
                      });
                    }
                  }}
                />
              </span>
              <p>Mark as done</p>
            </div>
          </section>
        {/if}
      </div>
      <div class="card-footer text-xs">
        {#if !isSummary}
          <p class="text-zinc-800 text-opacity-25">Created on: {getDateAsString(new Date(goal.createdAt))}</p>
          <p>Target Date: {getDateAsString(new Date(goal.targetDate))}</p>
        {:else}
          <p class="text-center">
            {viewId === goal.id ? 'Viewing ⚽' : 'Click to View More'}
          </p>
        {/if}
      </div>
    {:else}
      <!-- new goal -->
      <div class="card-header">
        New Goal
        {#if newGoal?.urgent || goal?.urgent}
          <span class="urgent-tag bg-yellow-100 text-yellow-800 text-xs font-semibold mr-2 px-2.5 py-0.5 rounded dark:bg-yellow-200 dark:text-yellow-900">URGENT</span>
        {/if}
      </div>
      <div class="card-body">
        <form id="newGoal" on:submit|preventDefault={handleCreate}>
          <div class="app-input">
            <label required for="objective">Objective</label>
            <input
              name="objective"
              autocomplete="off"
              type="text"
              placeholder="Objective"
              required
              bind:value={newGoal.objective}
            />
          </div>
          <div class="app-input">
            <label required for="estimatedHours">Estimated Total Hours Needed</label>
            <input
              type="number"
              min="1"
              name="estimatedHours"
              placeholder="Estimated Total Hours Needed"
              required
              bind:value={newGoal.estimatedHours}
            />
          </div>
          <div class="app-input">
            <label required for="targetDate">Target Date</label>
            <input
              type="date"
              name="targetDate"
              min={today.toLocaleDateString('en-ca')}
              placeholder="Target Date"
              required
              bind:value={newGoal.targetDate}
            />
          </div>
  
          <div class="app-input">
            <label for="motivations">What Motivates You to Achieve This? (optional)</label>
            <input
              type="text"
              autocomplete="off"
              name="motivations"
              placeholder="What Motivates You to Achieve This?"
              bind:value={newGoal.motivation}
            />
          </div>

          <div class="flex items-center mb-4 app-input">
            <span>
              <button
                type="button"
                class="circle toggle-urgent mr-2 app-border"
                aria-label="Mark as urgent"
                on:click={() => {
                  newGoal.urgent = !newGoal.urgent;
                }}
              />
            </span>
            <p>Mark as urgent</p>
          </div>

        </form>
      </div>
      <div class="card-footer">
        <button class="btn w-full" form="newGoal" type="submit"> Add Goal </button>
      </div>
    {/if}
  </div>
</div>

<style>
  .card {
    height: 100%;
    position: relative;
  }
  .goal-updates {
    display: inline-block;
    position: relative;
    width: 100%;
    margin-top: 1rem;
  }
  .goal-updates:before {
    position: absolute;
    content: '';
    border-top: 1px solid #a1a1a1c3;
    width: 70%;
    transform: translateX(-50%);
    left: 50%;
  }
  .goal-updates div:first-child {
    margin-top: 1rem;
  }
  progress[value]::-webkit-progress-value {
    background-image:
     -webkit-linear-gradient(top, rgba(255, 255, 255, .25), rgba(80, 80, 80, 0.25)),
     -webkit-linear-gradient(left, var(--success-color), var(--success-color));
}
</style>
