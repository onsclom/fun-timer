<script lang="ts">
	import { goto } from '$app/navigation';

	let minutes = 60;
	let seconds = 0;

	function start() {
		goto(`/run?minutes=${minutes}&seconds=${seconds}`);
	}
</script>

<h1>Fun timer ⏰</h1>

<form on:submit|preventDefault={start}>
	<label>
		Minutes
		<input type="number" min="0" max="99" bind:value={minutes} />
	</label>

	<label>
		Seconds
		<input type="number" min="0" max="59" bind:value={seconds} />
	</label>

	<input type="submit" value="Start" />
</form>

<style>
</style>
