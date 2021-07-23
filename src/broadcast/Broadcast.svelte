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

    <canvas use:setupCanvas width={config.width*config.resolution} height={config.height*config.resolution} />
    <canvas use:setupOutput width={config.width*config.resolution} height={config.height*config.resolution} />

    {#if isStarted}
        <div class="display" style="background-color:rgb({r},{g},{b});"></div>
    {/if}

    <div>

        {#if !isBroadcasting}
            <button on:click={start}>Start</button>
            <button on:click={()=>{ isSelected = false }}>Remove image</button>
        {:else}
            <button on:click={stop}>Stop</button>
        {/if}

        <p>
            {config.width} x {config.height} px = {config.width * config.height} area
            {#if isStarted}
                <br />
                progress {i} / {config.width * config.height}<br />
                R {r} G {g} B {b}
            {/if}
        </p>

    </div>

{/if}

<style>
	canvas {
        border: 1px solid #333;
    }
    .display {
        border: 1px solid #333;
        width: 250px;
        height: 250px;
        display: inline-block;
    }
    section {
        display: flex;
        flex-direction: column;
    }
    section > div {
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
</style>