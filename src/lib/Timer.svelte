<script lang="ts">
	let inSession = false;
  const originalMinuteTime = 25;
	let minute = originalMinuteTime;
	let sec = 0;

	function toggleTimer() {
			inSession = !inSession;
			if (inSession) {
				startTimer();
			} else {
        clearInterval(intervalId);
      }
	}

  let intervalId: any;
	function startTimer() {
		intervalId = setInterval(() => {
      sec--;
      if (sec < 1) {
        minute--;
        sec = 59;
      }
      if (minute === -1) {
        resetTimer();
      }
		}, 1000);
	}

  function resetTimer() {
    clearInterval(intervalId);
    inSession = false;
    minute = originalMinuteTime;
    sec = 0;
  }
</script>

<div class="card">
	<div class="card-header">Pomodoro Timer</div>
	<div class="card-body">
		<h1>
			{minute} : {sec < 10 ? '0' + sec : sec}
		</h1>
	</div>
	<div class="card-footer">
		<div class="w-full flex">
			{#if inSession}
				<button class="btn" on:click={toggleTimer}>Stop</button>
			{:else}
				<button class="btn" on:click={toggleTimer}>Start</button>
			{/if}
			<button class="btn" on:click={resetTimer}>Reset</button>
		</div>
	</div>
</div>

<style>
	.card {
		background-color: #d95550;
	}
	.card * {
		color: white;
	}
	.card .btn {
		color: #8f221e;
		background-color: white;
	}
</style>
