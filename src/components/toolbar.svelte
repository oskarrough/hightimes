<script>
	let {loop, started, paused, time} = $props()
</script>

<menu class="Toolbar">
	{#if !started}
		<button autofocus disabled={started} onclick={() => loop.start()}>Play High Times</button>
	{:else}
		<button disabled={!started} onclick={() => loop.stop()}>Stop Game</button>
		<button disabled={!loop.started || paused} onclick={() => loop.pause()}> Pause </button>
		<button disabled={!paused} onclick={() => loop.play()}> Resume </button>
		<span class="monospace">﹩{loop.datamodel.money} ⏱{time.toFixed(2)}ms</span>
	{/if}
</menu>

<style>
	.Toolbar {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 1;
		background: black;
		color: white;
		color-scheme: dark;
		border-bottom: 1px solid #ccc;
		padding: 0 4px;
		margin: 0;
		min-height: 2rem;
		display: flex;
		align-items: center;
	}

	span {
		margin-left: auto;
	}
</style>
