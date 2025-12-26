<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	/**
	 * Virtual Windchime Component
	 * 
	 * Plays random chime sounds at configurable intervals using the Web Audio API.
	 * Uses a pentatonic scale for pleasant, harmonious sounds.
	 * 
	 * Features:
	 * - Adjustable frequency (1-20 seconds average between chimes)
	 * - Start/Stop controls
	 * - Proper audio context initialization with user gesture handling
	 * - Memory leak prevention with cleanup on component destroy
	 */

	// State variables
	let frequency = 5; // Average seconds between chimes (configurable via slider)
	let audioContext: AudioContext | null = null; // Web Audio API context
	let isPlaying = false; // Whether chimes are currently playing
	let chimeTimeout: number | null = null; // Timeout ID for next chime
	let audioInitialized = false; // Whether audio context has been initialized

	/**
	 * Component lifecycle: Mount
	 * 
	 * Note: We don't auto-start audio here because modern browsers require
	 * a user gesture (click, tap, etc.) to initialize the AudioContext.
	 * Audio initialization happens when the user clicks "Start".
	 */
	onMount(() => {
		// Audio will be initialized on user interaction (Start button click)
	});

	/**
	 * Component lifecycle: Destroy
	 * 
	 * Clean up resources to prevent memory leaks:
	 * - Stop any pending chime timeouts
	 * - Close the audio context if it's still open
	 */
	onDestroy(() => {
		stopChiming();
		if (audioContext && audioContext.state !== 'closed') {
			audioContext.close();
		}
	});

	/**
	 * Initialize the Web Audio API context
	 * 
	 * Creates an AudioContext and ensures it's in a running state.
	 * This must be called in response to a user gesture (click, tap, etc.)
	 * due to browser autoplay policies.
	 * 
	 * @throws {Error} If AudioContext creation fails
	 */
	async function initializeAudio() {
		if (audioInitialized) return;
		
		try {
			// Create AudioContext (with fallback for older browsers)
			audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
			
			// Resume if suspended (browsers may suspend audio contexts)
			if (audioContext.state === 'suspended') {
				await audioContext.resume();
			}
			
			audioInitialized = true;
		} catch (error) {
			console.error('Failed to initialize audio context:', error);
		}
	}

	/**
	 * Start playing chimes
	 * 
	 * Initializes audio if needed and begins the chime sequence.
	 * This function is called when the user clicks the "Start" button.
	 */
	async function startChiming() {
		if (isPlaying) return;
		
		// Initialize audio context (requires user gesture)
		await initializeAudio();
		
		if (!audioContext) {
			console.error('Audio context not initialized');
			return;
		}
		
		isPlaying = true;
		scheduleNextChime();
	}

	/**
	 * Stop playing chimes
	 * 
	 * Stops the chime sequence and clears any pending timeouts.
	 * This function is called when the user clicks the "Stop" button.
	 */
	function stopChiming() {
		isPlaying = false;
		if (chimeTimeout !== null) {
			clearTimeout(chimeTimeout);
			chimeTimeout = null;
		}
	}

	/**
	 * Schedule the next chime
	 * 
	 * Calculates a random delay based on the frequency setting and
	 * schedules the next chime. The delay is randomized up to 2x the average
	 * to create a more natural, wind-like effect.
	 */
	function scheduleNextChime() {
		if (!isPlaying) return;
		
		// Random delay: 0 to 2x the average frequency
		const delay = Math.random() * frequency * 2;
		
		chimeTimeout = window.setTimeout(() => {
			playChime();
			scheduleNextChime(); // Schedule the next chime after this one plays
		}, delay * 1000);
	}

	/**
	 * Play a single chime sound
	 * 
	 * Creates and plays a chime sound using the Web Audio API.
	 * Uses a pentatonic scale for pleasant, harmonious sounds.
	 * 
	 * The sound envelope:
	 * - Attack: Quick rise to 0.3 volume (0.01s)
	 * - Decay: Exponential fade to near silence over 2 seconds
	 * 
	 * @throws {Error} If audio playback fails
	 */
	function playChime() {
		if (!audioContext || audioContext.state === 'closed') return;

		try {
			// Create oscillator (sound source) and gain node (volume control)
			const oscillator = audioContext.createOscillator();
			const gainNode = audioContext.createGain();

			// Connect nodes: oscillator -> gain -> output
			oscillator.connect(gainNode);
			gainNode.connect(audioContext.destination);

			// Pentatonic scale frequencies (C major pentatonic: C, D, E, G, A)
		// These notes always sound good together, creating a pleasant windchime effect
		const frequencies = [
				261.63, // C4 (Middle C)
				293.66, // D4
				329.63, // E4
				392.00, // G4
				440.00, // A4
				523.25  // C5 (High C)
			];
			
			// Select a random frequency from the pentatonic scale
			const freq = frequencies[Math.floor(Math.random() * frequencies.length)];
			oscillator.frequency.setValueAtTime(freq, audioContext.currentTime);

			// Audio envelope for natural chime sound
			// Start at 0 volume
			gainNode.gain.setValueAtTime(0, audioContext.currentTime);
			// Quick attack to 0.3 volume (10ms)
			gainNode.gain.linearRampToValueAtTime(0.3, audioContext.currentTime + 0.01);
			// Exponential decay to near silence over 2 seconds
			gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 2);

			// Play the sound
			oscillator.start(audioContext.currentTime);
			oscillator.stop(audioContext.currentTime + 2);
		} catch (error) {
			console.error('Error playing chime:', error);
		}
	}

	/**
	 * Handle frequency slider change
	 * 
	 * Updates the frequency variable when the user moves the slider.
	 * 
	 * @param event - The input event from the slider
	 */
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
