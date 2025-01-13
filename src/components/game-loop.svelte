<script>
	import {Loop} from 'vroum'
	import Toolbar from '../components/toolbar.svelte'
	import SomethingInteractive from '../components/something-interactive.svelte'
	import HistoryLog from './history-log.svelte'

	// These are mirrors of the loop's state, but tracked by Svelte, which makes sure the template is always re-rendered.
	let started = $state(false)
	let paused = $state(false)
	let time = $state(0)

	class MyLoop extends Loop {
		history = $state([])

		log = (message) => this.history.push({created: new Date().getTime(), message})

		begin() {
			started = true
			this.log('Begin')
		}
		destroy() {
			this.log('Destroy')
			started = false
		}
		$play = () => {
			paused = false
		}
		$pause = () => {
			paused = true
		}
		tick() {
			// by 1) updating local component state every tick and 2) rendering {time} in the template,
			// we ensure that the component is re-rendered every tick
			time = this.Loop.elapsedTime
		}
	}

	const loop = MyLoop.new()

	$effect(() => {
		console.log(loop)
	})
</script>

<Toolbar {loop} {started} {paused} {time} />

<HistoryLog {loop} />

{#if started}
	<SomethingInteractive {loop} />
{/if}
