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
		<button on:click={()=>{ mode = 'broadcast' }}>Broadcast</button>
		<button on:click={()=>{ mode = 'record' }}>Record</button>
	</nav>

{/if}

<style>
</style>