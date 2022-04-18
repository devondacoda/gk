<script lang="ts">
  import type { GoalType } from './Goal.svelte';
  export let goal: GoalType = null;
  export let type: 'motivation'|'currentStatus'|'timeTilTarget'|'' = ''
  const headers = {
    motivation: 'Your Motivation',
    currentStatus: 'Status',
    timeTilTarget: 'Time Left based on Estimate'
  };
  let daysLeft: number = null;
  let daysNeeded: number = null;
  let remainingHours: number = null;
  let hoursPerDay = 2;

  $: {
    if (goal) {
      const currDate = new Date();
      const targetDate = new Date(goal.targetDate);
      currDate.setUTCHours(0, 0, 0, 0);
      targetDate.setUTCHours(24, 0, 0, 0);
      const msLeft = targetDate.getTime() - currDate.getTime();
      daysLeft = Math.ceil(msLeft / (1000*60*60*24));
      remainingHours = (goal.estimatedHours - goal.hoursComplete);
      daysNeeded = remainingHours / hoursPerDay;
    }
  }
  $: doneStatus = (
    (type === 'currentStatus' && goal?.done) ||
    (type === 'timeTilTarget' && (remainingHours === 0 || goal?.done)) ||
    (type === 'motivation')
  )

  function pluralize(word: string, condition: boolean, extraLetter = 's') {
    return condition ? word + extraLetter : word;
  }
</script>

{#if goal && type}
  <div class="card" class:overdue={type === 'currentStatus' && daysLeft < 0} class:done={doneStatus}>
    <div class="card-header">
      {headers[type]}
    </div>
    {#if type === 'currentStatus'}
      {#if goal.done}
        <h2>Completed!</h2>
      {:else if daysLeft > 0}
        <p>Days left: {daysLeft}</p>
      {:else if daysLeft === 0}
        <h4>Today is the day!</h4>
      {:else}
        <h4>Overdue by {Math.abs(daysLeft)} {pluralize('day', Math.abs(daysLeft) !== 1)}</h4>
      {/if}
    {:else if type === 'motivation'}
      <p>{goal.motivation || 'Why not?'}</p>
    {:else if type === 'timeTilTarget'}
      {#if goal.done && daysLeft > 0}
        <p>You've finished {daysLeft} {pluralize('day', daysLeft !== 1)} ahead of schedule!</p>
      {:else if goal.done && daysLeft === 0 && remainingHours}
        <p>You've finished with {remainingHours} {pluralize('hour', remainingHours !== 1)} to spare!</p>
      {:else if remainingHours === 0}
        <p>All hours completed!</p>
      {:else}
        <p>
          You have {remainingHours} {pluralize('hour', remainingHours !== 1)} of your estimated total hours left.
        </p>
        <p>
          With <b>{hoursPerDay} hours</b> per day,
          you'd reach your total estimated hours
          {#if daysNeeded === 0}
            <b>today!</b>
          {:else}
            in <b>{daysNeeded} {pluralize('day', daysNeeded !== 1)}</b>
          {/if}
        </p>
      {/if}
    {/if}
  </div>
{/if}

<style>
  .card {
    height: 100%;
    text-align: center;
  }
  .overdue {
    border: 2px solid var(--warning-color);
  }
  .done {
    border: 2px solid var(--success-color);
  }
</style>
