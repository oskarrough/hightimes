<script>
	let {loop} = $props()
	let element

	function formatDateTime(timestamp) {
		const date = new Date(timestamp)
		const time = date.toLocaleTimeString('en-US', {
			hour12: false,
			hour: '2-digit',
			minute: '2-digit',
			second: '2-digit'
		})
		const ms = Math.floor(date.getMilliseconds() / 100) // Get tenths of a second
		const dateStr = date.toLocaleDateString({
			month: 'short',
			day: 'numeric'
		})
		return `${dateStr} ${time}.${ms}`
	}

	$effect(() => {
		// Scroll to bottom whenever history changes
		if (element && loop.history.length > 0) {
			element.scrollTop = element.scrollHeight
		}
	})
</script>

<ul class="history" bind:this={element}>
	{#each loop.history as record}
		<li><time>{formatDateTime(record.created)}</time>: {record.message}</li>
	{:else}
		<li>Welcome!</li>
	{/each}
</ul>

<style>
	.history {
		font-size: 13px;
		height: 8em;
		overflow-y: scroll;
		border: 1px solid;
		list-style: none;
		scroll-behavior: smooth;
		padding: 2px;
	}
	time {
		opacity: 0.8;
	}
</style>
