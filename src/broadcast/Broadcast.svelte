<script>

    import { onMount } from 'svelte';
    import { config } from '../config.js';

    /*
    variables
    */

    let isSelected = false;
    let isBroadcasting = false;
    let hasStarted = false;
    let hasEnded = false;

    let interval, image,
        inputCanvas, inputCtx, inputImageData, inputData,
        outputCanvas, outputCtx, outputImageData, outputData;

    let r = 0;
    let g = 0;
    let b = 0;

    let i = 0;
    let length = 0;

    const defaultImages = [
        'test-32.png',
        'earth-64.png',
        'flowers-128.png',
        'fabio-256.png',
        'lenna-512.png',
    ];
    let selectedImage = 'upload';
    let buttonText = 'Transmit';

    /*
    setup
    */

    onMount(()=>{
        // selectImage( defaultImages[1] );
    });

    function setupCanvas( element ){
        inputCtx = element.getContext("2d");
        inputCtx.mozImageSmoothingEnabled = false;
        inputCtx.imageSmoothingEnabled = false;
    }
    function setupOutput( element ){
        outputCtx = element.getContext("2d");
        outputCtx.mozImageSmoothingEnabled = false;
        outputCtx.imageSmoothingEnabled = false;
    }

    /*
    select image
    */

    function uploadImage(){
        if (!this.files || !this.files[0]) return;

        const FR = new FileReader();
        FR.addEventListener("load", (evt) => {
            inputCtx.clearRect( 0, 0, config.width, config.height );

            image = new Image();
            image.src = evt.target.result;
            image.addEventListener("load", () => {
                inputCtx.drawImage(image, 0, 0, config.width, config.height);
            });
        });
        FR.readAsDataURL(this.files[0]);

        isSelected = true;
    }

    function selectImage( filename, setMode = true ){
        if( filename === 'upload' ) return;
        
        image = new Image();
        image.onload = function() {
            inputCtx.drawImage(image, 0, 0, config.width, config.height);
        };
        image.src = `/images/${filename}`;
        console.log( image );

        isSelected = setMode;
    }
    function handleSelectImage( event ){
        selectImage( event.target.value );
    }

    /*
    broadcasting
    */

    function getProgress(){
        let progress = Math.round( i / outputData.length * 1000 ) / 10;
        document.title = `Transmitting ${progress}%`;
    }

    function broadcast(){
        interval = setInterval(() => {
            
            r = inputData[i];
            g = inputData[i+1];
            b = inputData[i+2];

            outputData[i] = r;
            outputData[i+1] = g;
            outputData[i+2] = b;
            outputData[i+3] = 255;

            outputCtx.putImageData(outputImageData, 0, 0);

            i += 4;

            getProgress();
            
            if( i > outputData.length ){
                clearInterval( interval );
                hasEnded = true;
            }
        }, 100);
    }

    /*
    controls
    */

    function play(){
        hasStarted = true;
        isBroadcasting = true;
        hasEnded = false;
    
        inputImageData = inputCtx.getImageData(0, 0, config.width, config.height);
        inputData = inputImageData.data;

        outputImageData = outputCtx.getImageData(0, 0, config.width, config.height);
        outputData = outputImageData.data;

        broadcast();
    }

    function handlePlay(){
        const now = Date.now();
        const startAt = Math.round( ( now + 1000 ) / 2000 ) * 2000;
        const startIn = startAt - now;
        buttonText = "Wait...";
        setTimeout(() => {
            play();
        }, startIn);
        console.log( now, startAt, startIn );
    }

    function pause(){
        buttonText = 'Resume';
        isBroadcasting = false;
        clearInterval( interval );
        outputCtx.putImageData(outputImageData, 0, 0);
    }

    function reset(){
        buttonText = 'Transmit';
        i = 0;
        outputCtx.clearRect(0, 0, config.width, config.height);
        hasStarted = false;
        r = 0;
        g = 0;
        b = 0;
    }

    function handleKeydown( event ){
        if( isSelected && event.keyCode === 32 ){
            if( isBroadcasting ){
                pause();
            } else {
                handlePlay();        
            }
        }
    }

</script>

<svelte:window on:keydown={handleKeydown}/>

{#if !isSelected}
    <main class="select">
        <div>
            <p>Please select an image or upload a file</p>
        </div>
        <div>
            <div class="select-wrapper">
                <select bind:value={selectedImage} on:change={handleSelectImage}>
                    {#each defaultImages as image}
                        <option value={image}>{image}</option>
                    {/each}
                    <option value='upload'>Upload</option>
                </select>
            </div>
            {#if selectedImage === 'upload'}
                <input type="file" on:change={uploadImage} accept="image/*" />
            {/if}
        </div>
    </main>
{:else}
    <main class="broadcast">
        
        <section class="input grey">
            <div>
                <p>{i/4} of {config.width * config.height} frames</p>
            </div>
            <div class="stretch">
                <canvas use:setupCanvas width={config.width*config.resolution} height={config.height*config.resolution} />
                <canvas use:setupOutput width={config.width*config.resolution} height={config.height*config.resolution} />
            </div>
            <div>
                <p style="padding: 0.5em;">&nbsp;</p>
            </div>
        </section>

        <section style="background-color:rgb({r},{g},{b});">
            <div>
                <p>
                    <span class="minispace">R</span>{r}
                    <span class="minispace">G</span>{g}
                    <span class="minispace">B</span>{b}
                </p>
            </div>
            <div>
                {#if !isBroadcasting}
                    {#if !hasEnded}
                        <button on:click={handlePlay}>{buttonText}</button>
                    {/if}
                    {#if hasStarted}
                        <button on:click={reset}>Reset</button>
                    {/if}
                {:else}
                    <button on:click={pause}>Stop</button>
                {/if}
            </div>
        </section>

    </main>
{/if}

<style>

    main.select {
        flex-direction: column;
    }
    main.select > div {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }

    select, input {
        display: block;
    }

    section.input .stretch {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
</style>