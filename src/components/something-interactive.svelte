<script>
	import {randomFromArray} from '../utils'
	import ProductTable from './product-table.svelte'
	import {Task} from 'vroum'

	let {loop = $bindable()} = $props()

	$effect(() => {
		loop.add(Populator.new(), Telegram.new())
	})

	let showAddConsumerPrompt = $state(true)

	let consumers = $state([])

	let datamodel = $derived(loop.datamodel)

	/** Available addictions */
	const addictionPool = ['Cough Syrup', 'Weed', 'Speed', 'LSD', 'Cocaine', 'Meth', 'Crack', 'Heroin']

	/** Returns a list of 1-3 random addictions */
	function randomAddictions() {
		const numberOfAddictions = Math.floor(Math.random() * 3) + 1
		return Array.from({length: numberOfAddictions}, () => randomFromArray(addictionPool))
	}

	function summarizeAddictions() {
		const summary = {}
		consumers.forEach((consumer) => {
			consumer.addictions.forEach((addiction) => {
				summary[addiction] = (summary[addiction] || 0) + 1
			})
		})
		return Object.entries(summary)
	}

	function randomBetween(min, max) {
		return Math.floor(Math.random() * (max - min + 1) + min)
	}

	function addConsumer(count = 1, addictions = randomAddictions()) {
		for (let i = 0; i < count; i++) {
			const newConsumer = Consumer.new({delay: randomBetween(100, 2000), addictions})
			loop.add(newConsumer)
		}
	}

	/** Buy a product from the market */
	function buy(product, quantity = 1) {
		const inStock = datamodel.market.find((p) => p.name === product.name).quantity

		if (!inStock) {
			alert('Out of stock')
			return
		}
		// pay
		datamodel.money = datamodel.money - product.price * quantity
		// update market stock
		datamodel.market.find((p) => p.name === product.name).quantity = inStock - quantity
		// add to inventory
		const purchasePrice = product.price * quantity
		datamodel.inventory[product.name] = {
			purchasePrice,
			quantity: (datamodel.inventory[product.name]?.quantity || 0) + quantity
		}
		loop.log(`Bought ${quantity}x ${product.name} for ${purchasePrice}`)
	}

	/** Sell a product from the inventory */
	function sell(productName, quantity = 1) {
		const inStock = datamodel.inventory[productName]?.quantity || 0
		if (!inStock) {
			this.Loop.log(`${productName} out of stock`)
			return
		}

		const product = datamodel.inventory[productName]

		// pay
		const price = marketPrice(product.purchasePrice)
		datamodel.money += price
		// update inventory stock
		product.quantity -= quantity
		loop.log(`Sold ${quantity}x ${productName} for ${price}`)
	}

	/**
	 * For now market price is hardcoded to double, but we can imagine a more complex formula, fluctuation, individual prices per product etc.
	 * @param normalPrice
	 */
	function marketPrice(normalPrice) {
		return normalPrice * 2
	}

	class Telegram extends Task {
		interval = 10000

		tick() {
			if (datamodel.messages.length > 1) {
				datamodel.messages.shift()
			}
		}
	}

	/** Represents someone looking to buy products */
	class Consumer extends Task {
		duration = 0
		interval = 6000
		failedAttempts = 0

		addictions = []

		begin() {
			this.Loop.log('Consumer added', this.addictions)
			consumers = loop.queryAll(Consumer)
		}

		destroy() {
			this.Loop.log('Consumer removed')
			consumers = loop.queryAll(Consumer)
		}

		tick() {
			if (this.failedAttempts > 2) {
				datamodel.messages.push('WTF - I am out of here!')
				this.disconnect()
				return
			}

			this.attemptToPurchase()
		}

		attemptToPurchase(amount = 1) {
			const productName = randomFromArray(this.addictions)
			const inStock = datamodel.inventory[productName]?.quantity || 0
			if (inStock) {
				sell(productName, amount)
				this.failedAttempts = 0
				this.Loop.log(`Consumer purchased ${amount} ${productName}`)
			} else {
				this.failedAttempts++
				this.Loop.log(`Consumer failed to purchase ${amount} of ${productName}. ${inStock} available`)
				datamodel.messages.push(`${productName} out of stock! Consumer failed ${this.failedAttempts} time(s).`)
			}
		}
	}

	class Populator extends Task {
		delay = 600
		duration = 0
		interval = 10000

		tick() {
			addConsumer(1, randomAddictions())
		}
	}

	let x = $derived(
		loop.history.filter((msg) => msg.message.includes('failed to purchase')).map((x) => x.message).length
	)
</script>

<!--// Dealer Interactions and weird Proposals-->
{#if showAddConsumerPrompt}
	<div class="message-box">
		<p>Florian: I'm coming to party with 20 friends. Got LSD?</p>
		<button onclick={() => addConsumer(20, ['LSD'])}>Na klar bruder</button>
		<button onclick={() => (showAddConsumerPrompt = false)}>Naah</button>
	</div>
{/if}

<div class="split">
	<div class="left">
		<div class="box">
			<header>
				<h2>Markt</h2>
			</header>
			<main>
				<ProductTable products={datamodel.market} money={datamodel.money} {buy} />
			</main>
		</div>
	</div>
	<div class="right-arrow"></div>
	<div class="right">
		<div class="box">
			<header>
				<h2>Inventory</h2>
			</header>
			<main>
				<ul class="list">
					{#each Object.entries(datamodel.inventory) as [name, product]}
						<li>{product.quantity}x {name}</li>
					{/each}
				</ul>
			</main>
		</div>
	</div>
</div>

{#snippet advertise(kiez, amount, addictions, cost)}
	<button disabled={datamodel.money < cost} onclick={() => addConsumer(amount, addictions)}>{kiez} {cost}﹩</button>
{/snippet}

<div class="box">
	<header><h2>Kiez030_Advertizing</h2></header>
	<main>
		<menu>
			{@render advertise('Görli', 1, ['Weed'], 20)}
			{@render advertise('Ostbahnhof', 3, ['Meth', 'Speed'], 60)}
			{@render advertise('Hauptbahnhof', 3, ['Cocaine'], 70)}
			{@render advertise('Technische Universität', 4, ['LSD', 'Cough Syrup'], 70)}
			{@render advertise('Berghain', 4, ['LSD', 'Cocaine', 'Meth'], 200)}
		</menu>
	</main>
</div>

<section>
	<div class="box">
		<header>
			<h2>Telegram ({consumers.length} members)</h2>
		</header>
		<main class="">
			<ul class="list" style="max-width: 15em">
				{#each summarizeAddictions() as [addiction, count]}
					<li>{addiction}: {count} consumer(s)</li>
				{/each}
			</ul>
			<div class="messages">
				<div class={{message: true, empty: !datamodel.currentMessage}}>
					<!-- {datamodel.messages.at(-1) || 'No new messages'} -->
				</div>
			</div>
		</main>
		<!-- svelte-ignore a11y_distracting_elements -->
		<marquee>{datamodel.messages.join(', ')}</marquee>
	</div>
</section>

<p>You've missed out on {x} orders.</p>

<!-- <section>
	<h2>Consumers</h2>
	<ul class="list">
		{#each consumers as consumer}
			<li>{consumer.addictions}</li>
		{/each}
	</ul>
</section> -->

<style>
	.box {
		margin: 1em 0;
		border: 1px solid;
		background-color: #faf9f9;
	}
	.box main {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		padding: 0 1rem;
	}
	.box header h2 {
		margin: 0;
		padding: 0 0.1rem;
		border-bottom: 1px solid;
	}

	.messages {
		flex: 1;
		padding: 1rem;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.message {
		background-color: white;
		padding: 1rem;
		text-align: center;
	}

	.message.empty {
		color: #333;
		background: none;
		box-shadow: none;
		font-style: italic;
	}

	.message-box {
		position: fixed;
		bottom: 20px;
		right: 20px;
		background-color: black;
		color: lime;
		font-family: 'Courier New', Courier, monospace;
		font-size: 1.2rem;
		border: 2px solid lime;
		padding: 15px;
		border-radius: 5px;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2);
		z-index: 1000;
		text-align: center;
		max-width: 30rem;
	}

	.message-box p {
		margin: 0 0 10px;
	}

	@keyframes fadeout {
		0% {
			opacity: 1;
		}
		100% {
			opacity: 0;
		}
	}
</style>
