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

    function upload(){
        if (!this.files || !this.files[0]) return;

        const FR = new FileReader();
        FR.addEventListener("load", (evt) => {
            inputCtx.clearRect( 0, 0, config.width, config.height );

            image = new Image();
            image.src = evt.target.result;
            image.addEventListener("load", () => {
                
                // let source = config.width * config.resolution;

                inputCtx.drawImage(image, 0, 0, config.width, config.height);
                // inputCtx.drawImage( image, 0, 0, xxx, xxx, 0, 0, yyy, yyy );
                // inputCtx.drawImage(image, 0, 0, source, source, 0, 0, target, target);
                
            });
        });
        FR.readAsDataURL(this.files[0]);

        isSelected = true;
    }

    function start(){
        isStarted = true;
        isBroadcasting = true;

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
        i = 0;
        outputCtx.putImageData(outputImageData, 0, 0);
    }

</script>

{#if !isSelected}
    <p>Please select an image</p>
    <input type="file" on:change={upload} accept="image/*" />
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

        {#if isStarted}
            <p>R {r} G {g} B {b}</p>
            <p>{i/4}/{250*250}</p>
        {/if}

    </div>

{/if}

<style>
	canvas {
        border: 1px solid #ddd;
    }
    .display {
        border: 1px solid #ddd;
        width: 250px;
        height: 250px;
        display: inline-block;
    }
</style>