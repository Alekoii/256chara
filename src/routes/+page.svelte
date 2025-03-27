<script lang="ts">
	import Button from '$components/Button.svelte';
	import { supabase } from '$lib/supabaseClient';

	interface JournalEntry {
		id: string;
		content: string;
		created_at: string;
	}

	const MAX_CHARS = 256;
	const ENTRIES_PER_PAGE = 15;

	let journal_entries: JournalEntry[] = $state([]);
	let newEntryContent = $state('');
	let charactersRemaining = $derived(MAX_CHARS - newEntryContent.length);
	let isLoading = $state(false);
	let hasMoreEntries = $state(true);
	let lastCreatedAt = $state<string | null>(null);
	let isSubmitting = $state(false);
	let totalEntries = $state<number | null>(null);

	async function fetchJournalEntries(loadMore = false) {
		if (isLoading) return;

		isLoading = true;

		try {
			let query = supabase
				.from('journal_entries')
				.select('id, content, created_at', { count: 'exact' })
				.order('created_at', { ascending: false })
				.limit(ENTRIES_PER_PAGE);

			if (loadMore && lastCreatedAt) {
				query = query.lt('created_at', lastCreatedAt);
			}

			const { data, error, count } = await query;

			if (error) {
				return;
			}

			if (!loadMore && count !== null) {
				totalEntries = count;
			}

			if (data) {
				if (data.length === 0 || data.length < ENTRIES_PER_PAGE) {
					hasMoreEntries = false;
				}

				if (data.length > 0) {
					lastCreatedAt = data[data.length - 1].created_at;
				}

				journal_entries = loadMore ? [...journal_entries, ...data] : data;
			}
		} finally {
			isLoading = false;
		}
	}

	async function loadMoreEntries() {
		await fetchJournalEntries(true);
	}

	async function addJournalEntry() {
		if (!newEntryContent.trim() || newEntryContent.length > MAX_CHARS) return;

		isSubmitting = true;

		const { data, error } = await supabase
			.from('journal_entries')
			.insert([{ content: newEntryContent }])
			.select();

		alert('Entry added successfully!');

		if (error) {
			console.error('Error adding entry:', error);
			return;
		}

		if (data) {
			journal_entries = [...data, ...journal_entries];
			newEntryContent = '';
		}
		isSubmitting = false;
	}

	function handleInput(e: Event) {
		const target = e.target as HTMLTextAreaElement;
		if (target.value.length > MAX_CHARS) {
			newEntryContent = target.value.slice(0, MAX_CHARS);
		}
	}

	let initialized = $state(false);

	$effect(() => {
		if (!initialized) {
			initialized = true;
			fetchJournalEntries();
		}
	});
</script>

<h1 class="heading">256 Chara</h1>
<p class="app-description">
	Practice writing in the language you're learning. Express yourself in just 256 characters. Short,
	focused practice builds confidence and fluency!
</p>

<div class="entry-form">
	<div class="textarea-container">
		<textarea
			bind:value={newEntryContent}
			placeholder="Write your 256 chara here..."
			rows="4"
			maxlength={MAX_CHARS}
			oninput={handleInput}
		></textarea>
		<div class="character-counter" class:warn={charactersRemaining < 20}>
			{charactersRemaining}
		</div>
	</div>
	<Button
		onclick={addJournalEntry}
		disabled={newEntryContent.length === 0 || newEntryContent.length > MAX_CHARS || isSubmitting}
	>
		{isSubmitting ? 'Saving...' : 'Save Entry'}
	</Button>
</div>

<div class="entries-container">
	{#if totalEntries !== null}
		<p class="total-count">Total Entries: {totalEntries}</p>
	{/if}
	<h2>Latest Entries</h2>
	{#each journal_entries as entry (entry.id)}
		<div class="entry">
			<p class="entry-content">{entry.content}</p>
			<p class="entry-time">{new Date(entry.created_at).toLocaleString()}</p>
		</div>
	{/each}

	{#if isLoading}
		<div class="loading">Loading entries...</div>
	{/if}

	{#if hasMoreEntries && !isLoading}
		<div class="load-more-container">
			<Button onclick={loadMoreEntries}>Load More</Button>
		</div>
	{/if}

	{#if !hasMoreEntries && journal_entries.length > 0}
		<div class="end-message">You've reached the end</div>
	{/if}

	{#if journal_entries.length === 0 && !isLoading}
		<div class="empty-message">No entries yet. Be the first to write something!</div>
	{/if}
</div>

<style>
	.entries-container {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		width: 600px;
		margin: 0 auto;
		padding-bottom: 2rem;
	}

	.entry {
		display: flex;
		gap: 0.25rem;
		flex-direction: column;
		padding: 0.5rem;
		border-left: 3px solid black;
	}

	.entry-content {
		font-size: 14px;
		overflow-wrap: break-word;
		text-wrap: balance;
	}

	.entry-time {
		font-size: 12px;
		color: grey;
	}

	.total-count {
		font-size: 14px;
		color: grey;
	}

	.entry-form {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
		margin: 0 auto;
		width: 600px;
		margin-bottom: 48px;
	}

	.textarea-container {
		position: relative;
	}

	.character-counter {
		position: absolute;
		bottom: 8px;
		right: 8px;
		font-size: 12px;
		color: #666;
		background-color: rgba(255, 255, 255, 0.8);
		padding: 2px 5px;
		border-radius: 3px;
	}

	.character-counter.warn {
		color: #d33;
		font-weight: bold;
	}

	.app-description {
		width: 600px;
		margin: 0 auto;
		font-size: 14px;
	}

	p,
	h2 {
		margin: 0;
	}

	.heading {
		margin: 0 auto;
		width: 600px;
	}

	textarea {
		padding: 0.5rem;
		border-radius: 4px;
		border: none;
		height: 100px;
		resize: none;
		width: 100%;
		box-sizing: border-box;
		border: 1px solid #ccc;
		font-size: 14px;
		margin-top: 14px;
		&:focus {
			outline: 1px solid black;
		}
	}
	.loading,
	.end-message,
	.empty-message {
		text-align: center;
		padding: 1rem;
		color: #666;
		font-style: italic;
	}

	.load-more-container {
		display: flex;
		justify-content: center;
		margin-top: 1rem;
	}
	@media only screen and (max-width: 600px) {
		.entry-form,
		.entries-container,
		.heading,
		.app-description {
			width: 100%;
		}
	}
</style>
