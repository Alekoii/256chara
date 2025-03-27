<script lang="ts">
	import Button from '$components/Button.svelte';
	import { supabase } from '$lib/supabaseClient';

	interface JournalEntry {
		content: string;
		created_at: string;
	}

	const MAX_CHARS = 256;

	let journal_entries: JournalEntry[] = $state([]);
	let newEntryContent = $state('');
	let charactersRemaining = $derived(MAX_CHARS - newEntryContent.length);

	async function fetchJournalEntries() {
		const { data } = await supabase
			.from('journal_entries')
			.select('content, created_at')
			.order('created_at', { ascending: false });
		if (data) journal_entries = data;
	}

	$effect(() => {
		fetchJournalEntries();
	});

	async function addJournalEntry() {
		if (!newEntryContent.trim() || newEntryContent.length > MAX_CHARS) return;

		const { data } = await supabase
			.from('journal_entries')
			.insert([{ content: newEntryContent }])
			.select();

		if (data) {
			await fetchJournalEntries();
			newEntryContent = '';
		}
	}

	function handleInput(e: Event) {
		const target = e.target as HTMLTextAreaElement;
		if (target.value.length > MAX_CHARS) {
			newEntryContent = target.value.slice(0, MAX_CHARS);
		}
	}
</script>

<h1 class="heading">256 Chara</h1>

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
		disabled={newEntryContent.length === 0 || newEntryContent.length > MAX_CHARS}
	>
		Save Entry
	</Button>
</div>

<div class="entries-container">
	<h2>Latest Entries</h2>
	{#each journal_entries as entry}
		<div class="entry">
			<p class="entry-content">{entry.content}</p>
			<p class="entry-time">{new Date(entry.created_at).toLocaleString()}</p>
		</div>
	{/each}
</div>

<style>
	.entries-container {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		width: 600px;
		margin: 0 auto;
	}

	.entry {
		display: flex;
		gap: 0.25rem;
		flex-direction: column;
		border-radius: 4px;
	}

	.entry-content {
		font-size: 14px;
	}

	.entry-time {
		font-size: 12px;
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

		&:focus {
			outline: none;
		}
	}

	@media only screen and (max-width: 600px) {
		.heading {
			width: 100%;
		}
		.entry-form {
			width: 100%;
		}
		.entries-container {
			width: 100%;
		}
	}
</style>
