<script lang="ts">
	import { onMount } from 'svelte';

	// Reactive state
	let word = $state<string>('');
	let canvas = $state<HTMLCanvasElement>();
	let result = $state<{
		imageUrl: string;
		letters: LetterInfo[];
		word: string;
	} | null>(null);
	let loading = $state<boolean>(false);
	let downloading = $state<boolean>(false);

	type Manifest = Record<string, string[]>;
	let manifest = $state<Manifest>({});

	const BASE_URL =
		'https://cdn.statically.io/gh/ReddyKousic/sky_glyphs_satellite_letters_webp@main';

	interface LetterInfo {
		letter: string;
		place: string;
		country: string;
		filename: string;
	}

	onMount(async () => {
		try {
			const res = await fetch(`${BASE_URL}/manifest.json`);
			manifest = await res.json();
		} catch (error) {
			console.error('Error loading manifest:', error);
		}
	});

	function parseFileName(filename: string): Omit<LetterInfo, 'filename'> {
		// Remove .webp extension and split by hyphens
		const parts = filename.replace('.webp', '').split('-');

		if (parts.length >= 3) {
			const letter = parts[0];

			// Remove trailing numeric parts from filename
			let countryIndex = parts.length - 1;
			while (countryIndex > 0 && /^\d+$/.test(parts[countryIndex])) {
				countryIndex--;
			}

			const country = parts[countryIndex];
			const place = parts.slice(1, countryIndex).join(' ');

			return {
				letter: letter.toUpperCase(),
				place: place
					? place.charAt(0).toUpperCase() + place.slice(1).replace(/([A-Z])/g, ' $1')
					: 'Unknown',
				country: country.charAt(0).toUpperCase() + country.slice(1)
			};
		}

		return { letter: '', place: 'Unknown', country: 'Unknown' };
	}

	async function generateWord() {
		if (!word.trim()) return;

		loading = true;
		result = null;

		const letters = word.toLowerCase().split('');
		const letterData: LetterInfo[] = [];
		const images: HTMLImageElement[] = [];

		for (const letter of letters) {
			const options = manifest[letter];
			if (!options || options.length === 0) continue;

			const randomFile = options[Math.floor(Math.random() * options.length)];
			const locationInfo = parseFileName(randomFile);

			const img: HTMLImageElement | null = await new Promise((resolve) => {
				const image = new Image();
				image.crossOrigin = 'anonymous';
				image.onload = () => resolve(image);
				image.onerror = () => resolve(null);
				image.src = `${BASE_URL}/${randomFile}`;
			});

			if (img) {
				images.push(img);
				letterData.push({
					...locationInfo,
					filename: randomFile
				});
			}
		}

		if (images.length > 0 && canvas) {
			const dataURL = await createImageFromLetters(images, canvas);
			result = {
				imageUrl: dataURL,
				letters: letterData,
				word: word.toUpperCase()
			};
		}

		loading = false;
	}

	async function createImageFromLetters(
		images: HTMLImageElement[],
		canvasEl: HTMLCanvasElement
	): Promise<string> {
		const scaleFactor = 0.75;

		// Calculate total width and max height
		const totalWidth = images.reduce((sum, img) => sum + img.width * scaleFactor, 0);
		const maxHeight = Math.max(...images.map((img) => img.height * scaleFactor));

		canvasEl.width = totalWidth;
		canvasEl.height = maxHeight;

		const ctx = canvasEl.getContext('2d');
		if (!ctx) throw new Error('Canvas context not available');

		// Draw all letter images
		let xOffset = 0;
		images.forEach((img) => {
			const scaledWidth = img.width * scaleFactor;
			const scaledHeight = img.height * scaleFactor;
			ctx.drawImage(img, xOffset, 0, scaledWidth, scaledHeight);
			xOffset += scaledWidth;
		});

		// Add watermark
		const watermarkText = 'https://skyglyphs.koastec.com/';
		const fontSize = Math.floor(canvasEl.width * 0.015); // 2.5% of width
		ctx.font = `${fontSize}px sans-serif`;
		ctx.textAlign = 'right';
		ctx.textBaseline = 'bottom';

		// Measure text for background rectangle
		const textMetrics = ctx.measureText(watermarkText);
		const padding = 6;
		const rectWidth = textMetrics.width + padding * 2;
		const rectHeight = fontSize + padding * 2;

		// Draw semi-transparent background rectangle
		ctx.fillStyle = 'rgba(0, 0, 0, 0.4)'; // dark background, 40% opacity
		ctx.fillRect(
			canvasEl.width - rectWidth - 10,
			canvasEl.height - rectHeight - 10,
			rectWidth,
			rectHeight
		);

		// Draw watermark text on top
		ctx.fillStyle = 'rgba(255, 255, 255, 0.2)'; // bright text, 90% opacity
		ctx.fillText(watermarkText, canvasEl.width - 10 - padding, canvasEl.height - 10 - padding / 2);

		return canvasEl.toDataURL('image/png');
	}

	async function downloadImage() {
		if (!result) return;

		downloading = true;

		try {
			// Add a small delay to show the loading state
			await new Promise((resolve) => setTimeout(resolve, 300));

			const link = document.createElement('a');
			link.download = `skyglyphs-${word}.png`;
			link.href = result.imageUrl;
			link.click();

			// Brief success state
			await new Promise((resolve) => setTimeout(resolve, 500));
		} catch (error) {
			console.error('Download failed:', error);
		} finally {
			downloading = false;
		}
	}

	function handleKeyPress(event: KeyboardEvent) {
		if (event.key === 'Enter') {
			generateWord();
		}
	}
</script>

<div class="min-h-screen bg-gray-50 p-8">
	<div class="mx-auto max-w-4xl">
		<!-- Header -->
		<div class="mb-12 text-center">
			<h1 class="mb-4 text-4xl font-bold text-gray-900">SkyGlyphs</h1>
			<p class="mx-auto max-w-2xl text-gray-600">
				Generate your name using satellite imagery from NASA's Landsat program
			</p>
			<p class="mt-2 text-xs text-gray-500 italic sm:text-sm">
				These are real satellite images — not AI generated. Collected from NASA's <a
					href="https://landsat.gsfc.nasa.gov/"
					class="text-green-600 hover:underline"
					target="_blank">Landsat</a
				> program and our growing community of contributors.
			</p>
		</div>

		<!-- Input -->
		<div class="mx-auto mb-12 flex w-full max-w-md flex-col justify-center gap-4 sm:flex-row">
			<input
				type="text"
				bind:value={word}
				onkeypress={handleKeyPress}
				placeholder="Enter your name"
				class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:ring-2 focus:ring-blue-500 focus:outline-none"
			/>
			<button
				onclick={generateWord}
				disabled={loading}
				class="flex min-h-[2.5rem] w-full items-center justify-center rounded-lg bg-green-600 px-6 py-2 whitespace-nowrap text-white transition-colors hover:bg-green-700 disabled:cursor-not-allowed disabled:opacity-75 sm:w-auto"
			>
				{#if loading}
					Generating...
				{:else if result}
					Generate New Variation
				{:else}
					Generate
				{/if}
			</button>
		</div>

		<!-- Loading -->
		{#if loading}
			<div class="mb-4 text-center">
				<div
					class="inline-block h-8 w-8 animate-spin rounded-full border-4 border-gray-300 border-t-green-600"
				></div>
				<p class="mt-4 text-gray-600">Generating your word...</p>
			</div>
		{/if}

		<!-- Result -->
		{#if result && !loading}
			<div class="mb-8 rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
				<!-- Generated Image -->
				<div class="mb-6 text-center">
					<img
						src={result.imageUrl}
						alt={result.word}
						class="mx-auto mb-4 max-w-full rounded border"
					/>
					<button
						onclick={downloadImage}
						disabled={downloading}
						class="inline-flex items-center gap-2 rounded-lg bg-gray-900 px-6 py-2 text-white transition-colors hover:bg-gray-800 disabled:cursor-not-allowed disabled:opacity-75"
					>
						{#if downloading}
							<div
								class="h-4 w-4 animate-spin rounded-full border-2 border-white border-t-transparent"
							></div>
							Downloading as PNG...
						{:else}
							Download Image
						{/if}
					</button>
				</div>

				<!-- Location Details -->
				<div class="border-t pt-6">
					<h3 class="mb-4 text-lg font-semibold text-gray-900">Locations Featured</h3>
					<div class="grid gap-4">
						{#each result.letters as letterInfo}
							<div class="flex items-center justify-between rounded bg-gray-50 px-4 py-2">
								<div class="flex items-center gap-4">
									<span class="text-2xl font-bold text-green-600">{letterInfo.letter}</span>
									<div>
										<p class="font-medium text-gray-900">{letterInfo.place}</p>
										<p class="text-sm text-gray-600">{letterInfo.country}</p>
									</div>
								</div>
							</div>
						{/each}
					</div>
				</div>
			</div>
		{/if}

		<div class="mb-8 rounded-lg border-1 border-green-600 bg-green-50 p-4 lg:text-left">
			<h3 class="mb-2 text-lg font-semibold text-gray-900">Help Expand SkyGlyphs!</h3>
			<p class="mb-4 text-gray-600">
				Have satellite images that look like letters? Contribute to our growing collection of
				Earth's natural alphabet.
			</p>
			<a
				href="https://github.com/ReddyKousic/sky_glyphs_satellite_letters_webp"
				target="_blank"
				rel="noopener noreferrer"
				class="inline-flex items-center rounded-lg bg-green-600 px-4 py-2 text-white transition-colors hover:bg-green-700"
			>
				🌍 Contribute Images on GitHub
			</a>
		</div>
		<!-- Credits / Community Footer -->
		<div class="space-y-3 border-t border-gray-200 pt-6 text-sm text-gray-500">
			<!-- Inspiration -->
			<p class="text-xs text-gray-400">
				Inspired by NASA's
				<a
					href="https://science.nasa.gov/specials/your-name-in-landsat/"
					class="text-green-600 hover:underline"
					target="_blank"
				>
					"Your Name in Landsat"
				</a>
			</p>
			<p class="text-xs text-gray-400">
				Original project by Ross Walter, Allison Nussbaum & Ginger Butcher (NASA Landsat Team)
			</p>

			<!-- Community -->
			<p class="mt-12 text-center text-gray-600">
				SkyGlyphs is community-driven — anyone can contribute satellite images.
			</p>

			<!-- Your Signature -->
			<div class="mt-4 flex items-center justify-center gap-3">
				<span class="h-px w-12 bg-gray-300"></span>
				<span class="text-base font-semibold text-gray-900">
					With 💚 from <a
						href="https://x.com/thKousic"
						target="_blank"
						rel="noopener noreferrer"
						class="text-[#7cb342] hover:underline">thKousic</a
					>
				</span>
				<span class="h-px w-12 bg-gray-300"></span>
			</div>
			<p class="text-center text-xs text-gray-500">Designer & Builder of this website</p>
		</div>
	</div>
</div>

<canvas bind:this={canvas} class="hidden"></canvas>
