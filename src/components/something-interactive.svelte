<script>
	import {randomFromArray} from '../utils'
	import ProductTable from './product-table.svelte'
	import {Task} from 'vroum'

	let {loop} = $props()

	$effect(() => {
		loop.add(Populator.new())
	})

	let messageQueue = []
	let isDisplayingMessage = $state(false)
	let showAddConsumerPrompt = $state(true)
	let money = $state(666)
	let datamodel = $state({
		currentMessage: null,
		inventory: {},
		market: [
			{
				name: 'Cough Syrup',
				quantity: 200,
				price: 5
			},
			{
				name: 'Weed',
				quantity: 150,
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

	let consumers = $state([])

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

	function addConsumer(count = 1, addictions = randomAddictions()) {
		for (let i = 0; i < count; i++) {
			const newConsumer = Consumer.new({addictions})
			loop.add(newConsumer)
		}
	}

	/** Buy a product from the market */
	function buy(product, quantity = 1) {
		// pay
		money = money - product.price * quantity
		// update market stock
		datamodel.market.find((p) => p.name === product.name).quantity--
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
		money += price
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

	function showMessage(text) {
		messageQueue.push(text)
		if (!isDisplayingMessage) {
			processMessageQueue()
		}
	}

	function processMessageQueue() {
		if (messageQueue.length === 0) {
			isDisplayingMessage = false
			return
		}

		isDisplayingMessage = true
		const nextMessage = messageQueue.shift()
		datamodel.currentMessage = nextMessage

		setTimeout(() => {
			datamodel.currentMessage = null
			processMessageQueue()
		}, 2000) // Display each message for 2 seconds
	}

	/** Represents someone looking to buy products */
	class Consumer extends Task {
		duration = 0
		interval = 4000
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
				showMessage('WTF - I am out of here!')
				this.disconnect()
				return
			}

			this.attemptToPurchase()
		}

		attemptToPurchase(amount = 1) {
			const productName = randomFromArray(this.addictions)
			const inStock = datamodel.inventory[productName]?.quantity || 0
			if (inStock) {
				console.log('in stock', productName, inStock)
				sell(productName, amount)
				this.failedAttempts = 0
				this.Loop.log(`Consumer purchased ${amount} ${productName}`)
			} else {
				this.failedAttempts++
				this.Loop.log(`Consumer failed to purchase ${amount} of ${productName}. ${inStock} available`)
				showMessage(`${productName} out of stock! Consumer failed ${this.failedAttempts} time(s).`)
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
</script>

<menu>
	<button onclick={() => addConsumer(1, ['Weed'])}>Görli</button>
	<button onclick={() => addConsumer(3, ['Meth', 'Speed'])}>Ostbahnhof</button>
	<button onclick={() => addConsumer(3, ['Cocaine'])}>Hauptbahnhof</button>
	<button onclick={() => addConsumer(4, ['LSD', 'Cough Syrup'])}>Technische Universität</button>
	<button onclick={() => addConsumer(4, ['LSD', 'Cocaine', 'Meth'])}>Berghain</button>
</menu>

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
		<h2>Markt</h2>
		<p>You have {money} moneys.</p>
		<ProductTable products={datamodel.market} {money} {buy} />
	</div>
	<div class="right-arrow"></div>
	<div class="right">
		<h2>Inventory</h2>
		<ul class="list">
			{#each Object.entries(datamodel.inventory) as [name, product]}
				<li>{product.quantity}x {name}</li>
			{/each}
		</ul>
	</div>
</div>

<section>
	<div id="message-screen">
		<h2>Telegram ({consumers.length} members)</h2>
		<div class="messages">
			<div class={{message: true, empty: !datamodel.currentMessage}}>
				{datamodel.currentMessage || 'No messages'}
			</div>
		</div>
	</div>

	<h2>Addictions Trends</h2>
	<ul class="list">
		{#each summarizeAddictions() as [addiction, count]}
			<li>{addiction}: {count} consumer(s)</li>
		{/each}
	</ul>
</section>

<section>
	<h2>Consumers</h2>
	<ul class="list">
		{#each consumers as consumer}
			<li>{consumer.addictions}</li>
		{/each}
	</ul>
</section>

<style>
	#message-screen {
		height: 200px;
		border: 1px solid;
		padding: 0 1rem;
		background-color: #e6e6e6;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		overflow: hidden;
		position: relative;
		margin-left: 0;
	}

	.messages {
		flex: 1;
		padding: 10px;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.message {
		background-color: white;
		padding: 10px;
		box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
		font-size: 1rem;
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
		left: 20px;
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
