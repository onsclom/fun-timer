<script lang="ts">
	import { onMount } from 'svelte';
	import { page } from '$app/stores';

	const mins = $page.url.searchParams.get('minutes');
	const secs = $page.url.searchParams.get('seconds');
	let timerAmount = (mins ? parseInt(mins) : 0) * 60 * 1000 + (secs ? parseInt(secs) : 0) * 1000;

	let canvas: HTMLCanvasElement;

	onMount(() => {
		let raf = 0;
		const start = performance.now();
		requestAnimationFrame(function update() {
			const now = performance.now();
			// TODO: figure out 1000 ms bug here
			const remainingTime = Math.max(1000, timerAmount - (now - start));

			const [minutes, seconds] = [
				Math.floor(remainingTime / 1000 / 60),
				Math.floor(remainingTime / 1000) % 60
			];

			canvas.width = window.innerWidth * devicePixelRatio;
			canvas.height = window.innerHeight * devicePixelRatio;
			const ctx = canvas.getContext('2d')!;
			ctx.fillStyle = 'black';
			ctx.fillRect(0, 0, canvas.width, canvas.height);

			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';
			ctx.fillStyle = 'white';

			const fontSize = Math.min(canvas.width, canvas.height) / 10;
			ctx.font = `${fontSize}px monospace`;

			const minsRemaining = Math.floor(remainingTime / 1000 / 60);
			const minString = String(minsRemaining).padStart(2, '0');

			const nextMinString = String(minsRemaining - 1).padStart(2, '0');
			const [firstMinDigit, secondMinDigit] = minString.split('');

			const timeTillNextMin = remainingTime % 60000;
			const minTransition = calculateTransition(timeTillNextMin);

			const timeTillNext10Min = remainingTime % 600000;
			const tenMinTransition = calculateTransition(timeTillNext10Min);

			const secondsRemaining = remainingTime / 1000;
			const secondString = String(Math.floor(secondsRemaining) % 60).padStart(2, '0');
			const [firstSecondDigit, secondSecondDigit] = secondString.split('');

			const nextSecondString = String(Math.floor(secondsRemaining - 1) % 60).padStart(2, '0');
			const [firstNextSecondDigit, secondNextSecondDigit] = nextSecondString.split('');

			const timeTillNextSecond = remainingTime % 1000;
			const secondTransition = calculateTransition(timeTillNextSecond);

			const timeTillNext10Second = remainingTime % 10000;
			const tenSecondTransition = calculateTransition(timeTillNext10Second);

			const scrollDisplay = [
				{
					cur: minString[0],
					next: nextMinString[0],
					transition: tenMinTransition
				},
				{
					cur: minString[1],
					next: nextMinString[1],
					transition: minTransition
				},
				{
					cur: ':',
					next: ':',
					transition: 0
				},
				{
					cur: firstSecondDigit,
					next: firstNextSecondDigit,
					transition: tenSecondTransition
				},
				{
					cur: secondSecondDigit,
					next: secondNextSecondDigit,
					transition: secondTransition
				}
			];

			const textWidth = ctx.measureText('0').width;
			const textHeight = fontSize;
			const digits = scrollDisplay.length;

			// lets draw each digit seperately now
			scrollDisplay.forEach((digit, i) => {
				const xOffset = i * textWidth;

				ctx.save();
				ctx.translate(
					canvas.width / 2 - (digits * textWidth) / 2 + xOffset,
					canvas.height / 2 + digit.transition * fontSize
				);
				{
					ctx.fillStyle = `rgba(255, 255, 255, ${1 - digit.transition})`;
					ctx.fillText(digit.cur, 0, 0);
					ctx.fillStyle = `rgba(255, 255, 255, ${digit.transition})`;
					ctx.fillText(digit.next, 0, -textHeight);
				}
				ctx.restore();
			});

			// const digit = Math.floor(secondsRemaining) % 10;
			// const nextDigit = Math.floor(secondsRemaining - 1) % 10;

			// const transitionAmount = calculateTransition(remainingTime);

			// ctx.save();
			// ctx.translate(0, transitionAmount * fontSize);
			// {
			// 	ctx.fillStyle = `rgba(255, 255, 255, ${1 - transitionAmount})`;
			// 	ctx.fillText(`${digit}`, canvas.width / 2, canvas.height / 2);
			// 	const textWidth = ctx.measureText(`${digit}`).width;
			// 	const textHeight = fontSize;
			// 	ctx.fillStyle = `rgba(255, 255, 255, ${transitionAmount})`;
			// 	ctx.fillText(`${nextDigit}`, canvas.width / 2, canvas.height / 2 - textHeight);
			// }
			// ctx.restore();

			// lets draw the seconds now

			raf = requestAnimationFrame(update);
		});

		return () => cancelAnimationFrame(raf);
	});

	function calculateTransition(t: number) {
		const remaining = t / 1000;
		const scaled = remaining / 0.25;
		const clamped = Math.min(1, Math.max(0, scaled));
		const inversed = 1 - clamped;
		return easeInOutQuad(inversed);
	}

	function easeInOutQuad(t: number) {
		return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
	}
</script>

<h1>timer running!</h1>

<canvas bind:this={canvas} />

<style>
	canvas {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}
</style>
