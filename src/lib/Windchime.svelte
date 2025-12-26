<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	let frequency = 5; // average seconds between chimes
	let audioContext: AudioContext | null = null;
	let isPlaying = false;
	let chimeTimeout: number | null = null;
	let audioInitialized = false;

	onMount(() => {
		// Don't auto-start - wait for user interaction
	});

	onDestroy(() => {
		stopChiming();
		if (audioContext && audioContext.state !== 'closed') {
			audioContext.close();
		}
	});

	async function initializeAudio() {
		if (audioInitialized) return;
		
		try {
			audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
			if (audioContext.state === 'suspended') {
				await audioContext.resume();
			}
			audioInitialized = true;
		} catch (error) {
			console.error('Failed to initialize audio context:', error);
		}
	}

	async function startChiming() {
		if (isPlaying) return;
		
		await initializeAudio();
		if (!audioContext) {
			console.error('Audio context not initialized');
			return;
		}
		
		isPlaying = true;
		scheduleNextChime();
	}

	function stopChiming() {
		isPlaying = false;
		if (chimeTimeout !== null) {
			clearTimeout(chimeTimeout);
			chimeTimeout = null;
		}
	}

	function scheduleNextChime() {
		if (!isPlaying) return;
		const delay = Math.random() * frequency * 2; // random up to 2x average
		chimeTimeout = window.setTimeout(() => {
			playChime();
			scheduleNextChime();
		}, delay * 1000);
	}

	function playChime() {
		if (!audioContext || audioContext.state === 'closed') return;

		try {
			const oscillator = audioContext.createOscillator();
			const gainNode = audioContext.createGain();

			oscillator.connect(gainNode);
			gainNode.connect(audioContext.destination);

			// Random frequency for chime (pentatonic scale for pleasant sound)
			const frequencies = [261.63, 293.66, 329.63, 392.00, 440.00, 523.25]; // C4, D4, E4, G4, A4, C5
			const freq = frequencies[Math.floor(Math.random() * frequencies.length)];
			oscillator.frequency.setValueAtTime(freq, audioContext.currentTime);

			// Envelope for natural chime sound
			gainNode.gain.setValueAtTime(0, audioContext.currentTime);
			gainNode.gain.linearRampToValueAtTime(0.3, audioContext.currentTime + 0.01);
			gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 2);

			oscillator.start(audioContext.currentTime);
			oscillator.stop(audioContext.currentTime + 2);
		} catch (error) {
			console.error('Error playing chime:', error);
		}
	}

	function handleFrequencyChange(event: Event) {
		const target = event.target as HTMLInputElement;
		frequency = parseFloat(target.value);
	}
</script>

<div class="p-4 max-w-md mx-auto">
	<h1 class="text-2xl font-bold mb-4">Virtual Windchime</h1>
	<div class="form-control">
		<label class="label" for="frequency-slider">
			<span class="label-text">Chime Frequency (avg seconds)</span>
		</label>
		<input
			id="frequency-slider"
			type="range"
			min="1"
			max="20"
			step="0.5"
			value={frequency}
			oninput={handleFrequencyChange}
			class="range range-primary"
		/>
		<div class="flex justify-between text-xs px-2">
			<span>1s</span>
			<span>20s</span>
		</div>
		<div class="text-center mt-2">Current: {frequency}s</div>
	</div>
	<div class="mt-4 flex gap-2">
		<button class="btn btn-primary" onclick={startChiming} disabled={isPlaying}>Start</button>
		<button class="btn btn-secondary" onclick={stopChiming} disabled={!isPlaying}>Stop</button>
	</div>
	{#if !audioInitialized && !isPlaying}
		<p class="text-xs text-gray-500 mt-2 text-center">Click Start to begin</p>
	{/if}
</div>