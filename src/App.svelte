<script>

	let mode = false;

	import Broadcast from './broadcast/Broadcast.svelte';
	import Record from './record/Record.svelte';

</script>

{#if mode === 'broadcast'}

	<Broadcast />

{:else if mode === 'record'}

	<Record />

{:else}

	<nav>
		<div>
			<button on:click={()=>{ mode = 'broadcast' }}>Broadcast</button>
		</div>
		<div class="white">
			<button on:click={()=>{ mode = 'record' }}>Record</button>
		</div>
	</nav>

{/if}

<style>

	nav {
		display: flex;
		width: 100vw;
		height: 100vh;
	}
	nav > div {
		flex: 1;
		display: flex;
	}
	button {
		display: block;
		flex: 1;
	}
	button:hover {
		animation: blink 200ms linear infinite;
	}

</style>