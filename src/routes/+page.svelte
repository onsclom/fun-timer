<script lang="ts">
	let minute = 10;
	let second = 0;
	let running = false;

	let remainingTime = 0;
	let raf: number;
	let paused = false;

	function startTimer() {
		remainingTime = minute * 60 + second;
		lastTime = performance.now();
		running = true;
		raf = requestAnimationFrame(timerLoop);
	}

	let lastTime = 0;
	function timerLoop(time: number) {
		const delta = time - lastTime;
		lastTime = time;

		if (!paused) {
			remainingTime -= delta / 1000;
			remainingTime = Math.max(0, remainingTime);
			minute = Math.floor((remainingTime % 3600) / 60);
			second = Math.floor(remainingTime % 60);
		}

		if (remainingTime != 0) raf = requestAnimationFrame(timerLoop);
	}

	function pad(n: number) {
		return n.toString().padStart(2, '0');
	}

	function reset() {
		running = false;
		cancelAnimationFrame(raf);
	}
</script>

<div
	style="display: flex; flex-direction: column; width: 100%; height: 100%;
justify-content: center; align-items: center; gap: 1rem"
>
	{#if running}
		<!-- playing view -->
		<div style="font-size: 2rem">
			{minute}:{pad(second)}.{pad(Math.floor((remainingTime % 1) * 100))}
		</div>

		<div style="display: flex; gap: 1rem">
			<button on:click={() => (paused = !paused)} disabled={remainingTime === 0}
				>{paused ? 'resume' : 'pause'}</button
			>
			<button on:click={reset}>edit</button>
		</div>
	{:else}
		<!-- setup view -->
		<form
			on:submit|preventDefault={startTimer}
			style="display: flex; flex-direction: column; gap: 1rem; display: flex; justify-content: center; align-items: center; gap: 1rem"
		>
			<div>
				<label>
					<input type="number" inputmode="numeric" size="2" min="0" max="99" bind:value={minute} />
					m
				</label>
				<label>
					<input type="number" inputmode="numeric" size="2" min="0" max="99" bind:value={second} />
					s
				</label>
			</div>
			<input type="submit" value="start" />
		</form>
	{/if}
</div>

<style>
	/* increase global font size */
	:global(html) {
		font-size: 2rem;
	}
</style>
