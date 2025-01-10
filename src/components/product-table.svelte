<script>
	import {products} from '../products.js'

	const {datamodel} = $props()

	function buy(product, quantity) {
		// pay
		datamodel.money = datamodel.money - product.price * quantity
		// add to inventory
		datamodel.inventory[product.name] = {
			quantity: (datamodel.inventory[product.name]?.quantity || 0) + quantity,
			purchasePrice: product.price
		}
	}
</script>

<ul>
{#each products as product}
	<li>
		<button onclick={() => buy(product, 1)}>Buy 1 {product.name}</button>
		{product.quantity} items - {product.price} EUR
	</li>
{/each}
</ul>
