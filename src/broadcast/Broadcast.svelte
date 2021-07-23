<script>

    import { onMount } from 'svelte';
    import { config } from '../config.js';

    let isSelected = false;
    let isBroadcasting = false;
    let isStarted = false;

    let interval, image,
        inputCanvas, inputCtx, inputImageData, inputData,
        outputCanvas, outputCtx, outputImageData, outputData;

    let r = 0;
    let g = 0;
    let b = 0;

    let i = 0;
    let length = 0;

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

    function start(){
        isStarted = true;
        isBroadcasting = true;
        i = 0;
        outputCtx.clearRect(0, 0, config.width, config.height);

        inputImageData = inputCtx.getImageData(0, 0, config.width, config.height);
        inputData = inputImageData.data;

        outputImageData = outputCtx.getImageData(0, 0, config.width, config.height);
        outputData = outputImageData.data;

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
        }, 100);
    }

    function stop(){
        isBroadcasting = false;
        clearInterval( interval );
        outputCtx.putImageData(outputImageData, 0, 0);
    }

    const defaultImages = [
        'earth-64.png',
        'flowers-128.jpg'
    ];
    let selectedImage = 'upload';
    onMount(()=>{
        // selectImage( defaultImages[1] );
    });

</script>

{#if !isSelected}
    <section class="select full">
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
    </section>
{:else}
    <section class="full broadcast">
        
        <div class="panel input">
            <div class="info">
                <p>{config.width}x{config.height} px</p>
            </div>
            <div class="main">
                <canvas use:setupCanvas width={config.width*config.resolution} height={config.height*config.resolution} />
                <canvas use:setupOutput width={config.width*config.resolution} height={config.height*config.resolution} />
            </div>
            <div class="info">
                <p>{i/4} of {config.width * config.height} frames</p>
            </div>
        </div>

        <div class="panel display" style="background-color:rgb({r},{g},{b});">
            <div><p>R{r} G{g} B{b}</p></div>
            <div>
                {#if !isBroadcasting}
                    <button on:click={start}>Start</button>
                {:else}
                    <button on:click={stop}>Stop</button>
                {/if}
            </div>
        </div>

    </section>
{/if}

<style>
	canvas {
        background-color: black;
        margin: 0.5rem;
    }
    section.select {
        flex-direction: column;
    }
    section.select > div {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }
    select, input {
        display: block;
        width: 50vw;
    }
    select {
        background-color: white;
    }
    section.broadcast > div {
        flex: 1;
    }
    .panel {
        display: flex;
        justify-content: space-between;
        flex-direction: column;
    }
    .panel > div {
        padding: 1rem;
    }
    .padding {
        padding-bottom: 0.5rem;
    }
    .panel.input {
        background-color: #222;
    }
    .panel.input .main {
        flex: 1;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
</style>