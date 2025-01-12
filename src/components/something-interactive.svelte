<script>
	import {randomFromArray} from '../utils'
	import ProductTable from './product-table.svelte'
	import {Task} from 'vroum'

	let {loop} = $props()

	let messageQueue = []
	let isDisplayingMessage = $state(false)
	let showAddConsumerPrompt = $state(true)
	let datamodel = $state({
		money: 666,
		currentMessage: null,
		inventory: {},
		products: [
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

	// List of possible addictions
	const addictionPool = ['Cough Syrup', 'Weed', 'Speed', 'LSD', 'Cocaine', 'Meth', 'Crack', 'Heroin']

	// Generate random addictions
	function randomAddictions() {
		const numberOfAddictions = Math.floor(Math.random() * 3) + 1 // 1 to 3 addictions
		return Array.from({length: numberOfAddictions}, () => randomFromArray(addictionPool))
	}

	function addConsumer(count = 1, addictions = randomAddictions()) {
		for (let i = 0; i < count; i++) {
			const newConsumer = Consumer.new({addictions})
			loop.add(newConsumer)
		}
	}

	function buy(product, quantity) {
		// pay
		datamodel.money = datamodel.money - product.price * quantity
		// update market stock
		datamodel.products.find((p) => p.name === product.name).quantity--
		// add to inventory
		datamodel.inventory[product.name] = {
			quantity: (datamodel.inventory[product.name]?.quantity || 0) + quantity,
			purchasePrice: product.price
		}
	}

	function handleAddConsumerPrompt(accept) {
		if (accept) {
			addConsumer(5)
		}
		showAddConsumerPrompt = false
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

	class Consumer extends Task {
		duration = 0
		interval = 3000
		failedAttempts = 0

		begin() {
			consumers = loop.queryAll(Consumer)
			console.log('Consumer added', this.addictions)
		}

		destroy() {
			consumers = loop.queryAll(Consumer)
			console.log('Consumer removed')
		}

		tick() {
			if (this.failedAttempts > 2) {
				showMessage('WTF - I am out of here!')
				this.disconnect()
				return
			}
			this.attemptToPurchase()
		}

		attemptToPurchase() {
			const productName = randomFromArray(this.addictions)
			console.log('Consumer wants', productName)

			const product = datamodel.inventory[productName]

			if (product?.quantity) {
				product.quantity -= 1
				datamodel.money += product.purchasePrice * 2
				this.failedAttempts = 0 // Reset failed attempts on success
			} else {
				showMessage(`${productName} out of stock! Consumer wanted ${this.failedAttempts} time(s).`)
				this.failedAttempts++
			}
		}
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
</script>

<h2>Add consumers</h2>
<button onclick={() => addConsumer(1, ['Weed'])}>Görli</button>
<button onclick={() => addConsumer(3, ['Meth', 'Speed'])}>Ostbahnhof</button>
<button onclick={() => addConsumer(3, ['Cocaine'])}>Hauptbahnhof</button>
<button onclick={() => addConsumer(4, ['LSD', 'Cough Syrup'])}>Technische Universität</button>
<button onclick={() => addConsumer(4, ['LSD', 'Cocaine', 'Meth'])}>Berghain</button>

<!--// Dealer Interactions and weird Proposals-->
{#if showAddConsumerPrompt}
	<div class="message-box">
		<p>Florian: I'm coming to party with 20 friends. You got LSD?</p>
		<button onclick={() => addConsumer(20, ['LSD'])}>Na Klar bruder</button>
		<button onclick={() => handleAddConsumerPrompt(false)}>No</button>
	</div>
{/if}

<div class="split">
	<div class="left">
		<h2>Markt</h2>
		<p>You have {datamodel.money} moneys.</p>
		<ProductTable {datamodel} {buy} />
	</div>
	<div class="right-arrow"></div>
	<div class="right">
		<h2>Inventory</h2>
		<ul>
			{#each Object.entries(datamodel.inventory) as [name, product]}
				<li>{product.quantity}x {name}</li>
			{/each}
		</ul>
	</div>
</div>

<h2>Your Telegram Channel - Members: {consumers.length}</h2>
<div id="message-screen">
	<div class="messages">
		<div class={{message: true, empty: !datamodel.currentMessage}}>
			{datamodel.currentMessage || 'No messages'}
		</div>
	</div>
</div>

<h2>Addictions Trends</h2>
<ul>
	{#each summarizeAddictions() as [addiction, count]}
		<li>{addiction}: {count} consumer(s)</li>
	{/each}
</ul>

<ul>
	{#each consumers as consumer}
		<li>{consumer.addictions}</li>
	{/each}
</ul>

<style>
	#message-screen {
		width: 300px;
		height: 200px;
		border: 2px solid #333;
		border-radius: 20px;
		background-color: #e6e6e6;
		box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		overflow: hidden;
		position: relative;
		font-family: Arial, sans-serif;
		margin-left: 0;
		/* margin: 20px auto; */
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
		border-radius: 10px;
		box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
		font-size: 1rem;
		text-align: center;
	}

	.message.empty {
		color: #999;
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

	.split {
		display: flex;
		margin: 1rem 0;
		gap: 0;
		align-items: center;
	}

	.left,
	.right {
		flex: 1;
		border: 1px solid;
		padding: 0 0.5rem;
	}

	.right-arrow {
		width: 0;
		height: 0;
		border-top: 1rem solid transparent;
		border-bottom: 1rem solid transparent;
		border-left: 1rem solid;
	}
</style>
