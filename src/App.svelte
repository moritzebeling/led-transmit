<script>
	let mode = 'broadcast';

	import Broadcast from './broadcast/Broadcast.svelte';
	import Record from './record/Record.svelte';
</script>

<nav>
	<button on:click={()=>{ mode = 'broadcast' }}>Broadcast</button>
	<button on:click={()=>{ mode = 'record' }}>Record</button>
</nav>

{#if mode === 'broadcast'}
	<Broadcast />
{:else if mode === 'record'}
	<Record />
{/if}

<style>
	
</style>