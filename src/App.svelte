<script>
	
	import { Router, Link, Route } from "svelte-routing";
	export let url = "";

	import Broadcast from './broadcast/Broadcast.svelte';
	import Record from './record/Record.svelte';

</script>

<Router url="{url}">

	<Route path="/">
		<nav class="full">
			<div>
				<Link to="broadcast">Broadcast</Link>
			</div>
			<div class="white">
				<Link to="record">Record</Link>
			</div>
		</nav>
	</Route>

	<Route path="broadcast" component="{Broadcast}" />
	<Route path="record" component="{Record}" />

</Router>
	
<style>

	@media (orientation: portrait) {
		nav {
			flex-direction: column;
		}
	}
	nav > div {
		flex: 1;
		display: flex;
		padding: 1rem;
		align-items: stretch;
		justify-content: stretch;
	}
	nav :global( a ) {
		background-color: transparent;
		color: inherit;
		display: block;
		width: 100%;
	}
	nav :global( a:hover ) {
		animation: blink 200ms linear infinite;
	}

</style>