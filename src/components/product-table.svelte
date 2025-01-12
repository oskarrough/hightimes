<script>
	const {datamodel, buy} = $props()
</script>

<ul>
	{#each datamodel.products as product}
		<li>
			<button disabled={datamodel.money < product.price * 1} onclick={() => buy(product, 1)}
				>Buy 1 {product.name}</button
			>
			{product.quantity} items - {product.price} EUR
		</li>
	{/each}
</ul>
