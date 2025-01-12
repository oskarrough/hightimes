<script>
	import {Loop} from 'vroum'
	import Toolbar from '../components/toolbar.svelte'
	import SomethingInteractive from '../components/something-interactive.svelte'

	// These are mirrors of the loop's state, but tracked by Svelte, which makes sure the template is always re-rendered.
	let started = $state(false)
	let paused = $state(false)
	let time = $state(0)

	class MyLoop extends Loop {
		begin() {
			started = true
		}
		destroy() {
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

	let loop = $state(MyLoop.new())

	$effect(() => {
		console.log('child', loop)
	})
</script>

<Toolbar {loop} {started} {paused} {time} />

{#if started}
	<SomethingInteractive {loop} />
{/if}
