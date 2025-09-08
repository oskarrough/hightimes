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

		datamodel = $state({
			gameover: false,
			money: 666,
			messages: [],
			currentMessage: null,
			inventory: {},
			market: [
				{
					name: 'Cough Syrup',
					quantity: 80,
					price: 5
				},
				{
					name: 'Weed',
					quantity: 60,
					price: 15
				},
				{
					name: 'Speed',
					quantity: 50,
					price: 25
				},
				{
					name: 'LSD',
					quantity: 30,
					price: 50
				},
				{
					name: 'Cocaine',
					quantity: 20,
					price: 100
				},
				{
					name: 'Meth',
					quantity: 15,
					price: 80
				},
				{
					name: 'Crack',
					quantity: 20,
					price: 60
				},
				{
					name: 'Heroin',
					quantity: 10,
					price: 120
				}
			]
		})

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
			this.checkWinCondition()
		}

		/** You win if the datamodel.market is completely out of stock. */
		checkWinCondition() {
			if (this.datamodel.market.every((product) => product.quantity === 0)) {
				this.log(`You win! ${this.datamodel.money}﹩ in ${(this.Loop.elapsedTime / 1000).toFixed(1)}s`)
				this.datamode.gameover = true
				this.pause()
			}
		}
	}

	let loop = $state(new MyLoop())

	$effect(() => {
		window.oskar = loop
	})
</script>

<Toolbar {loop} {started} {paused} {time} />

{#if started}
	<SomethingInteractive bind:loop />
	<HistoryLog bind:loop />
{/if}
