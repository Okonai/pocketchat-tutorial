<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import App from '../App.svelte';
  import { currentUser, pb } from './pocketbase';
	import { flip } from 'svelte/animate'

	const dragDuration = 300
	let cards: any[] = []
	let draggingCard: any
	let animatingCards = new Set<number>()
  let movelist = [];

  let newComboName = ''


	function swapWith(card) {
		if (draggingCard === card || animatingCards.has(card)) return
		animatingCards.add(draggingCard)
		setTimeout(() => animatingCards.delete(draggingCard), dragDuration)
		const cardAIndex = cards.indexOf(draggingCard)
		const cardBIndex = cards.indexOf(card)
		cards[cardAIndex] = card
		cards[cardBIndex] = draggingCard
  }

  let combos = [];
  let unsubscribe: () => void;
  let formOpen = false;

  onMount(async () => {
    const resultList = await pb.collection('combos').getList(1, 50, {
      sort: 'created',
      expand: 'user',
    });
    combos = resultList.items;

    const record = await pb.collection('fighters').getOne('ue66lsqf5rsi409', {
        expand: 'name, frame_data',
    });
    movelist = [...record.frame_data.normals, ...record.frame_data.specials]
    // Subscribe to realtime combos
    unsubscribe = await pb
      .collection('combos')
      .subscribe('*', async ({ action, record }) => {
        if (action === 'create') {
          // Fetch associated user
          const user = await pb.collection('users').getOne(record.user);
          record.expand = { user };
          combos = [...combos, record];
        }
        if (action === 'delete') {
          combos = combos.filter((m) => m.id !== record.id);
        }
      });
  });

  // Unsubscribe from realtime combos
  onDestroy(() => {
    unsubscribe?.();
  });

  async function createCombo( ) {
    const data = {
      name: newComboName,
      movelist: cards,
      user: $currentUser.id,
      setttings: {
        damage: 0,
        meter: 0,
        stun: 0,
        time: 0,
      },
    };
    const createdCombo = await pb.collection('combos').create(data);
    combos = [...combos, createdCombo];
    formOpen = false;
  }

  function addToCombo (move) {
    cards = [
      ...cards,
      {
        key: Math.random(), 
        ...move
      }
    ] 
  }

</script>

<!-- Combos component -->
<div class="combos">
  <h1>Combos</h1>
  <button on:click={() => formOpen = true}>Create Combo</button>
  {#each combos as combo (combo.id)}
    <div class="combo">
      <img
        class="avatar"
        src={`https://avatars.dicebear.com/api/identicon/${combo.expand?.user?.username}.svg`}
        alt="avatar"
        width="40px"
      />
      <div>
        <small>
          Sent by @{combo.expand?.user?.username}
        </small>
        <h2>{combo.name}</h2>
        <p>{combo.moves}</p>
      </div>
    </div>
  {/each}
</div>

<!-- Form component -->
{#if formOpen}
  <form on:submit|preventDefault={() => formOpen = false}>
    <label for="combo-name">Combo Name</label>
    <input type="text" id="combo-name" name="combo-name" bind:value={newComboName} required/>
    <div class="moveList">
      {#each movelist as move (move)}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div
          animate:flip={{ duration: dragDuration }}
          class="moves"
          on:click={() => addToCombo(move)}
        >
          {move.input}
        </div>
      {/each}
    </div>
    <label for="combo-moves">TIMELINE</label>
    <div class="combo-moves">
      {#each cards as card (card)}
        <span
          animate:flip={{ duration: dragDuration }}
          class="cards"
          draggable="true"
          on:dragstart={() => draggingCard = card}
          on:dragend={() => draggingCard = undefined}
          on:dragenter={() => swapWith(card)}
          on:dragover|preventDefault
        >
          {card.input}
        </span>
      {/each}
    </div>
    <button type="submit" on:click={() => createCombo()}>Save Combo</button>
    <button type="button">Cancel</button>
  </form>
{/if}


<style>
  /* 
    Flat minimalistic design, 
    You can play with blue, cyan, green, red, range faint background
    and white, black, gray, range faint text colors

    Shhould be responsvive, the moves in the movelist should be easz to drag and drop from
    the character moves to the combo moves and vice versa

    The combo moves container should flow from left to righh like text, but justified on both sides of the container
    combo move card should be around 16-24 px padding and no vertical margin opnly 2 pixel horizontal margin

    Take a look at this example: https://www.youtube.com/shorts/KXP9dws64uI
  */

  .combos {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
  }

  .combos h1 {
    font-size: 2rem;
    margin: 0;
  }

  .combos button {
    margin: 1rem;
  }

  .combos .combo {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    max-width: 600px;
    padding: 1rem;
    margin: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .combos .combo .avatar {
    margin-right: 1rem;
  }

  .combos .combo small {
    display: block;
    margin-bottom: 0.5rem;
  }

  .combos .combo h2 {
    margin: 0;
  }

  .combos .combo p {
    margin: 0;
  }

  .combos form {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    max-width: 600px;
    padding: 1rem;
    margin: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .combos form label {
    display: block;
    margin-bottom: 0.5rem;
  }

  .combos form input {
    margin-bottom: 1rem;
  }

  .combos form .combo-moves {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    width: 100%;
    max-width: 600px;
    padding: 1rem;
    margin: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .combos form .combo-moves .card {
    margin: 2px;
    padding: 16px;
    background: #ccc;
    border-radius: 4px;
  }

  .combos form .combo-moves .card:hover {
    background: #bbb;
  }

  .combo-moves {
    padding: 30px;
    border:#ADD8E6 1px solid;
  }

  .moveList {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    width: 100%;
    max-width: 600px;
    padding: 1rem;
    margin: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .moves {
    margin: 2px;
    padding: 4px;
    background: #ADD8E6;
    border-radius: 4px;
  }

  .moves:hover {
    background: #bbb;
  }

  .combo-moves .cards {
    margin: 2px;
    padding: 16px;
    background: #ccc;
    border-radius: 4px;
  }

  .combo-moves .cards:hover {
    background: #bbb;
  }

  .combo-moves .cards {
    margin: 2px;
    padding: 16px;
    background: #ccc;
    border-radius: 4px;
  }

  .combo-moves .cards:hover {
    background: #bbb;
  }

  input {
    width: 100%;
    padding: 12px 20px;
    margin: 8px 0;
    box-sizing: border-box;
  }
</style>

