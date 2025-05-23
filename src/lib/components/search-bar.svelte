<script lang="ts">
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { docsSearch } from './doc-search.svelte';
	import * as Command from '$lib/components/ui/command';
	import { Search as SearchIcon, BookOpen, FileText } from 'lucide-svelte';
	import Separator from './ui/separator/separator.svelte';

	let open = $state(false);
	let searchQuery = $state('');
	let searchResults = $derived(docsSearch.searchResults);

	$effect(() => {
		if (searchQuery) {
			docsSearch.search(searchQuery);
		}
	});

	function handleResultClick(slug: string) {
		searchQuery = '';
		open = false;
		docsSearch.clearSearch();
		goto(`/docs/${slug}`);
	}

	onMount(() => {
		function handleKeydown(e: KeyboardEvent) {
			if (e.key === 'k' && (e.metaKey || e.ctrlKey)) {
				e.preventDefault();
				open = !open;
			}
		}

		document.addEventListener('keydown', handleKeydown);

		return () => {
			document.removeEventListener('keydown', handleKeydown);
		};
	});

	function handleSearchClick() {
		open = true;
	}
</script>

<button
	class="flex items-center gap-2 transition-opacity hover:opacity-80"
	onclick={handleSearchClick}
>
	<div class="flex items-center gap-2">
		<SearchIcon class="h-4 w-4 text-muted-foreground" />
		<p class="hidden text-sm text-muted-foreground sm:block">Search documentation</p>
		<kbd
			class="pointer-events-none inline-flex h-5 select-none items-center gap-1 rounded border bg-muted px-1.5 font-mono text-[10px] font-medium text-muted-foreground opacity-100"
		>
			<span class="text-xs">⌘</span>K
		</kbd>
	</div>
</button>

<Command.Dialog bind:open>
	<div class="relative">
		<Command.Input bind:value={searchQuery} placeholder="Search documentation..." class="pl-10" />
	</div>

	<Command.List>
		{#if searchQuery === ''}
			<Command.Empty class="py-6 text-center text-sm">
				Start typing to search documentation...
			</Command.Empty>
		{:else if searchResults.length === 0}
			<Command.Empty class="py-6 text-center text-sm">
				No results found for "{searchQuery}"
			</Command.Empty>
		{:else if searchResults.length > 0}
			<Command.Group heading="Documentation">
				{#each searchResults as result}
					<Command.Item
						onSelect={() => handleResultClick(result.slug)}
						class="flex items-start gap-2 px-2 py-3"
					>
						<FileText class="mt-0.5 h-4 w-4 shrink-0" />
						<div class="flex flex-col gap-1">
							<span class="font-medium">{result.title}</span>
							<span class="line-clamp-2 text-sm text-muted-foreground">
								{result.description}
							</span>
						</div>
					</Command.Item>
				{/each}
			</Command.Group>
		{/if}

		{#if searchQuery === '' || searchResults.length > 0}
			<Command.Group heading="Quick Links">
				<Command.Item onSelect={() => handleResultClick('')}>
					<BookOpen class="mr-2 h-4 w-4" />
					<span>Browse All Documentation</span>
				</Command.Item>
			</Command.Group>
		{/if}
	</Command.List>
</Command.Dialog>
