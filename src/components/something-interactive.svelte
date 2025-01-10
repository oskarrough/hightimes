<script>
	import { randomFromArray } from '../utils'
import ProductTable from './product-table.svelte'
import {Loop, Task} from 'vroum'
const {what} = $props()

const datamodel = $state({
	money: 666,
	inventory: {},
})

class Consumer extends Task {
	duration = 0
	interval = 1000

	addictions = ['weed', 'speed']

	begin() {
		console.log('consumer addded?')
	}

	tick() {
		this.attemptToPurchase()
	}

	attemptToPurchase() {
		const productName = randomFromArray(this.addictions)
		console.log('buy something', productName)
		const product = datamodel.inventory[productName]
		if (product?.quantity > 0) {
			product.quantity -= 1
			datamodel.money += product.purchasePrice
		}
	}
}

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
loop.add(Consumer.new())
loop.add(Consumer.new({addictions: ['oranges']}))
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

<!-- <button onclick={clickit}>Click me {count}</button> -->

<h2>Market</h2>
<ProductTable {datamodel} />

<h2>Inventory</h2>
<p>Money = {datamodel.money}</p>
{#each Object.entries(datamodel.inventory) as [name, product]}
	<p>{product.quantity}x {name}</p>
{/each}

<h2>Consumers</h2>
<p>@todo visualize consumers and their addictions</p>
{#each datamodel.consumers as consumer}
	<p>X</p>
{/each}
