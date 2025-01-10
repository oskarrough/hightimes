<script>
	import {randomFromArray} from '../utils'
	import ProductTable from './product-table.svelte'
	import {Loop, Task} from 'vroum'

	let time = $state(0)
	let messageQueue = [] 
	let isDisplayingMessage = false 
	let showAddConsumerPrompt = true
	
	const {what} = $props()
	const datamodel = $state({
		money: 666,
		inventory: {},
		consumers: [],
		currentMessage: null
	})

	 

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
	
	// List of possible addictions
	const addictionPool = ['Cough Syrup', 'Weed', 'Speed', 'LSD', 'Cocaine', 'Meth', 'Crack', 'Heroin']

	// Generate random addictions
	function randomAddictions() {
		const numberOfAddictions = Math.floor(Math.random() * 3) + 1 // 1 to 3 addictions
		return Array.from({length: numberOfAddictions}, () => randomFromArray(addictionPool))
	}

	function addConsumer(count = 1, addictions = null) {
		for (let i = 0; i < count; i++) {
			const newConsumer = Consumer.new({addictions: addictions || randomAddictions()}) // Use provided addictions or generate random ones
			loop.add(newConsumer)
			datamodel.consumers.push(newConsumer)
		}
	}

	function removeConsumer(consumer) {
		const index = datamodel.consumers.indexOf(consumer)
		if (index !== -1) {
			datamodel.consumers.splice(index, 1) 
			loop.remove(consumer) 
			showMessage('WTF - I am out of here!')
			console.log('Consumer removed:', consumer)
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
	
	class MyLoop extends Loop {
		begin() {
			started = true
		}

		tick() {
			time = this.Loop.elapsedTime.toFixed(2)
		}
	}

	class Consumer extends Task {
		duration = 0
		interval = 3000
		failedAttempts = 0 // Counter for failed attempts

		constructor({addictions = []} = {}) {
			super()
			this.addictions = addictions
		}

		begin() {
			console.log('Consumer added with addictions:', this.addictions)
		}

		tick() {
			this.attemptToPurchase()
		}

		attemptToPurchase() {
			const productName = randomFromArray(this.addictions)
			console.log('Attempting to buy:', productName)

			const product = datamodel.inventory[productName]
			if (product?.quantity > 0) {
				product.quantity -= 1
				datamodel.money += product.purchasePrice * 2
				this.failedAttempts = 0 // Reset failed attempts on success
			} else {
				this.failedAttempts++
				showMessage(`${productName} out of stock! Consumer has failed ${this.failedAttempts} time(s).`)
				if (this.failedAttempts >= 3) {
					removeConsumer(this)
				}
			}
		}
	}

	let started = $state(false)
	let paused = $state(false)
	let count = $state(Number(what))
	let loop = MyLoop.new()

	function summarizeAddictions() {
		const summary = {}
		datamodel.consumers.forEach((consumer) => {
			consumer.addictions.forEach((addiction) => {
				summary[addiction] = (summary[addiction] || 0) + 1
			})
		})
		return Object.entries(summary)
	}
</script>

<menu>
	{#if !started}
		<button onclick={() => loop.start()}>Start loop</button>
	{:else if paused}
		<button onclick={() => play()}>Resume loop</button>
	{:else}
		<button onclick={() => pause()}>Pause loop</button>
	{/if}
</menu>


<p style="font-family: monospace">{time}ms</p>

<button onclick={() => addConsumer(1, ['Weed'])}>Görlitzer Park</button>
<button onclick={() => addConsumer(3, ['Meth', 'Speed'])}>OstBahnhof</button>
<button onclick={() => addConsumer(3, ['Cocaine'])}>Hauptbahnhof</button>
<button onclick={() => addConsumer(4, ['LSD', 'Cough Syrup'])}>Technische Universität</button>
<button onclick={() => addConsumer(4, ['LSD', 'Cocaine', 'Meth'])}>Berghain</button>

<!--// Dealer Interactions and weird Proposals-->
{#if showAddConsumerPrompt}
	<div class="message-box">
		<p>Florian: I'm coming to party with 20 friends. You got LSD? </p>
		<button onclick={() => addConsumer(20, ['LSD'])}>Na Klar bruder</button>
		<button onclick={() => handleAddConsumerPrompt(false)}>No</button>
	</div>
{/if}

<h2>Market</h2>
<ProductTable {datamodel} />

<h2>Inventory</h2>
<h3>Black Money = {datamodel.money}</h3>
{#each Object.entries(datamodel.inventory) as [name, product]}
	<p>{product.quantity}x {name}</p>
{/each}

<h2>Your Telegram Channel - Members: {datamodel.consumers.length}</h2>
<div id="message-screen">
	<div class="messages">
		{#if datamodel.currentMessage}
			<div class="message">{datamodel.currentMessage}</div>
		{:else}
			<div class="message empty">No messages yet...</div>
		{/if}
	</div>
</div>

<h2>Addiction Trends</h2>
<ul>
	{#each summarizeAddictions() as [addiction, count]}
		<li>{addiction}: {count} consumer(s)</li>
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
</style>
