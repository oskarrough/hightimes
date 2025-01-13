<script>
	const {products, money, buy} = $props()
</script>

<ul class="list">
	{#each products as product}
		<li>
			{product.name}
			<button disabled={product.quantity < 1 || money < product.price * 1} onclick={() => buy(product, 1)}>Buy 1</button
			>
			<button disabled={product.quantity < 5 || money < product.price * 5} onclick={() => buy(product, 5)}>Buy 5</button
			>
			{product.quantity} available
			<span class="price">{product.price}﹩</span>
		</li>
	{/each}
</ul>

<style>
	.price {
		float: right;
	}
</style>
