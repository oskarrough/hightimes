<script>
	let {loop} = $props()
	let element

	$effect(() => {
		// Scroll to bottom whenever history changes
		if (element && loop.history.length > 0) {
			element.scrollTop = element.scrollHeight
		}
	})
</script>

<ul class="history" bind:this={element}>
	{#each loop.history as record}
		<li>{record.created}: {record.message}</li>
	{:else}
		<li>Welcome!</li>
	{/each}
</ul>

<style>
	.history {
		font-size: 10px;
		height: 5em;
		overflow: auto;
		border: 1px solid;
		border-radius: 1rem;
		list-style: none;
		scroll-behavior: smooth;
	}
</style>
