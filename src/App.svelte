<script lang="ts">
  import Lists from "./components/Lists.svelte";
import Combos from "./lib/Combos.svelte";
  import Login from "./lib/Login.svelte";
  import Messages from "./lib/Messages.svelte";
  import { currentUser } from "./lib/pocketbase";


</script>

<h1>Mentor AI</h1>

<Login />

{#if $currentUser}
  <Combos />
{/if}

<!-- <Lists /> -->