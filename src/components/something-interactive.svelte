<script>
import {Loop, Task} from 'vroum'
const {what} = $props()

class MyLoop extends Loop {
	begin() {
		started = true
	}

	tick() {
		time = this.Loop.elapsedTime.toFixed(2)
	}
}

let started = $state(false)
let paused = $state(false)
let count = $state(Number(what))
let loop = MyLoop.new()
let time = $state(0)

function play() {
	loop.play()
	paused = false
}

function pause() {
	loop.pause()
	paused = true
}

function clickit() {
	count = count + 1
}
</script>

<menu>
	{#if !started}
		<button onclick={() => loop.start()}>Start loop</button>
	{:else}
		{#if paused}
			<button onclick={() => play()}>Resume loop</button>
		{:else}
			<button onclick={() => pause()}>Pause loop</button>
		{/if}
	{/if}
</menu>

<p style="font-family: monospace">{time}ms</p>

<button onclick={clickit}>Click me {count}</button>
