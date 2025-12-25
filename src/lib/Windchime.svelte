<script lang="ts">
	import { onMount } from 'svelte';

	let frequency = 5; // average seconds between chimes
	let audioContext: AudioContext | null = null;
	let isPlaying = false;

	onMount(() => {
		audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
		startChiming();
	});

	function startChiming() {
		if (isPlaying) return;
		isPlaying = true;
		scheduleNextChime();
	}

	function stopChiming() {
		isPlaying = false;
	}

	function scheduleNextChime() {
		if (!isPlaying) return;
		const delay = Math.random() * frequency * 2; // random up to 2x average
		setTimeout(() => {
			playChime();
			scheduleNextChime();
		}, delay * 1000);
	}

	function playChime() {
		if (!audioContext) return;

		const oscillator = audioContext.createOscillator();
		const gainNode = audioContext.createGain();

		oscillator.connect(gainNode);
		gainNode.connect(audioContext.destination);

		// Random frequency for chime
		const frequencies = [261.63, 293.66, 329.63, 349.23, 392.00, 440.00, 493.88]; // C4 to B4
		const freq = frequencies[Math.floor(Math.random() * frequencies.length)];
		oscillator.frequency.setValueAtTime(freq, audioContext.currentTime);

		// Envelope
		gainNode.gain.setValueAtTime(0, audioContext.currentTime);
		gainNode.gain.linearRampToValueAtTime(0.3, audioContext.currentTime + 0.01);
		gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 1);

		oscillator.start(audioContext.currentTime);
		oscillator.stop(audioContext.currentTime + 1);
	}

	function handleFrequencyChange(event: Event) {
		const target = event.target as HTMLInputElement;
		frequency = parseFloat(target.value);
	}
</script>

<div class="p-4 max-w-md mx-auto">
	<h1 class="text-2xl font-bold mb-4">Virtual Windchime</h1>
	<div class="form-control">
		<label class="label">
			<span class="label-text">Chime Frequency (avg seconds)</span>
		</label>
		<input
			type="range"
			min="1"
			max="20"
			step="0.5"
			value={frequency}
			on:input={handleFrequencyChange}
			class="range range-primary"
		/>
		<div class="flex justify-between text-xs px-2">
			<span>1s</span>
			<span>20s</span>
		</div>
		<div class="text-center mt-2">Current: {frequency}s</div>
	</div>
	<div class="mt-4 flex gap-2">
		<button class="btn btn-primary" on:click={startChiming} disabled={isPlaying}>Start</button>
		<button class="btn btn-secondary" on:click={stopChiming} disabled={!isPlaying}>Stop</button>
	</div>
</div>