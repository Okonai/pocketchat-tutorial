<script lang="ts">
	import { flip } from 'svelte/animate'

	const dragDuration = 300
	let cards: number[] = Array(20).fill(undefined).map((_, i) => i + 1)
	let draggingCard: any
	let animatingCards = new Set<number>()

	function swapWith(card: number) {
		if (draggingCard === card || animatingCards.has(card)) return
		animatingCards.add(draggingCard)
		setTimeout(() => animatingCards.delete(draggingCard), dragDuration)
		const cardAIndex = cards.indexOf(draggingCard)
		const cardBIndex = cards.indexOf(card)
		cards[cardAIndex] = card
		cards[cardBIndex] = draggingCard
	}
</script>

<div class="container">
	{#each cards as card (card)}
		<div
			animate:flip={{ duration: dragDuration }}
			class="card"
			draggable="true"
			on:dragstart={() => draggingCard = card}
			on:dragend={() => draggingCard = undefined}
		  on:dragenter={() => swapWith(card)}
			on:dragover|preventDefault
		>
			{card}
		</div>
	{/each}
</div>

<style>
	.container {
		display: grid;
		grid-template-rows: repeat(4, 1fr);
		grid-template-columns: repeat(5, 1fr);
		gap: 24px;
	}

	.card {
		display: flex;
		justify-content: center;
		align-items: center;
    color: darkblue;
		background-color: lightblue;
		width: 100%;
		height: 96px;
		font-size: 1.5rem;
	}
</style>