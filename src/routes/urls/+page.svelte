<script lang="ts">
	import { onMount } from 'svelte';
	import { writable, derived } from 'svelte/store';

	const BASE_URL =
		'https://cdn.statically.io/gh/ReddyKousic/sky_glyphs_satellite_letters_webp/main';

	type Manifest = Record<string, string[]>;

	const manifest = writable<Manifest>({});
	const allUrls = writable<string[]>([]);
	const chunkSize = 10;
	const currentPage = writable(0);

	const totalPages = derived(allUrls, ($allUrls) => Math.ceil($allUrls.length / chunkSize));

	const currentChunk = derived([allUrls, currentPage], ([$allUrls, $currentPage]) =>
		$allUrls.slice($currentPage * chunkSize, ($currentPage + 1) * chunkSize)
	);

	async function loadManifest() {
		try {
			const res = await fetch(`${BASE_URL}/manifest.json`);
			const data: Manifest = await res.json();
			manifest.set(data);

			// Flatten all URLs
			const urls: string[] = [];
			for (const letter in data) {
				for (const file of data[letter]) {
					urls.push(`${BASE_URL}/${file}`);
				}
			}
			allUrls.set(urls);
		} catch (err) {
			console.error('Failed to load manifest:', err);
		}
	}

	function copyChunk(chunk: string[]) {
		navigator.clipboard
			.writeText(chunk.join('\n'))
			.then(() => alert('Copied to clipboard!'))
			.catch(() => alert('Failed to copy.'));
	}

	function nextPage() {
		currentPage.update((p) => p + 1);
	}

	function prevPage() {
		currentPage.update((p) => p - 1);
	}

	onMount(() => {
		loadManifest();
	});
</script>

<div class="min-h-screen bg-gray-50 p-8">
	<div class="mx-auto max-w-4xl">
		<h1 class="mb-6 text-center text-3xl font-bold">All SkyGlyphs Image URLs</h1>

		{#if $currentChunk.length === 0}
			<p class="text-center text-gray-600">Loading URLs...</p>
		{:else}
			<div class="space-y-4">
				{#each $currentChunk as url}
					<div class="flex items-center justify-between rounded bg-white p-2 shadow">
						<span class="text-sm break-all">{url}</span>
					</div>
				{/each}
			</div>

			<div class="mt-4 flex justify-between">
				<button
					on:click={prevPage}
					class="rounded bg-gray-300 px-4 py-2 hover:bg-gray-400"
					disabled={$currentPage === 0}
				>
					Previous
				</button>
				<button
					on:click={() => copyChunk($currentChunk)}
					class="rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700"
				>
					Copy This Chunk
				</button>
				<button
					on:click={nextPage}
					class="rounded bg-gray-300 px-4 py-2 hover:bg-gray-400"
					disabled={$currentPage >= $totalPages - 1}
				>
					Next
				</button>
			</div>

			<p class="mt-2 text-center text-sm text-gray-500">
				Page {$currentPage + 1} of {$totalPages}
			</p>
		{/if}
	</div>
</div>
