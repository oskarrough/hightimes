<script>
	const {products, buy} = $props()
</script>

<ul>
	{#each products as product}
		<li>
			<button onclick={() => buy(product, 1)}>Buy 1 {product.name}</button>
			{product.quantity} items - {product.price} EUR
		</li>
	{/each}
</ul>
