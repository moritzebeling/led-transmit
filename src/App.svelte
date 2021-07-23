<script>
	
	import { Router, Link, Route } from "svelte-routing";
	export let url = "";

	import Broadcast from './broadcast/Broadcast.svelte';
	import Record from './record/Record.svelte';

</script>

<Router url="{url}">

	<Route path="/">
		<nav class="main">
			<section>
				<Link to="broadcast">Broadcast</Link>
			</section>
			<section class="white">
				<Link to="record">Record</Link>
			</section>
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
	section {
		align-items: stretch;
		justify-content: stretch;
		flex-direction: unset;
	}
	nav :global( a ) {
		padding: 1em;
		background-color: transparent;
		color: inherit;
		display: block;
		width: 100%;
	}
	nav :global( a:hover ) {
		animation: blink 200ms linear infinite;
	}

</style>