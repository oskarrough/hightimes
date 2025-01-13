<script>
	const {products, money, buy} = $props()
</script>

<ul class="list">
	{#each products as product}
		<li>
			<button disabled={money < product.price * 1} onclick={() => buy(product, 1)}
				>Buy 1 {product.name}</button
			>
			{product.quantity} items - {product.price} EUR
		</li>
	{/each}
</ul>
