<script>

    import { onMount } from 'svelte';

	function getFullHeight(){
		document.documentElement.style.setProperty('--full-height', `${Math.floor(window.innerHeight)}px`);
    }
    onMount(()=>{
        getFullHeight();
    });

</script>

<svelte:window on:resize={getFullHeight} />