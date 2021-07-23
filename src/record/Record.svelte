<script>

    import { onMount } from 'svelte';
    import { config } from '../config.js';

    /*
    variables
    */

    let video,
        inputCanvas, inputCtx, inputImageData, inputData,
        outputCanvas, outputCtx, outputImageData, outputData;

    let isRecording = false;
    let hasStarted = false;
    let hasEnded = false;

    let r = 0;
    let g = 0;
    let b = 0;

    let i = 0;

    let width = 16;
    let height = 16;

    let buttonText = "Record";

    /*
    setup
    */
    
    onMount(()=>{

        navigator.mediaDevices.getUserMedia({
            video: {
                width:     width,
                height:    height,
                frameRate: config.framerate
            }
        }).then(function(stream) {
            video.srcObject = stream;
            video.onloadedmetadata = function(e) {
                video.play();
            };
        }).catch(function(err) {
            // deal with an error (such as no webcam)
        });

        inputCanvas.width = config.width;
        inputCanvas.height = config.height;

        inputCtx = inputCanvas.getContext('2d');
        outputCtx = outputCanvas.getContext('2d');

        inputCtx.drawImage(video, 0, 0, width, height);
        inputCtx.mozImageSmoothingEnabled = false;
        inputCtx.imageSmoothingEnabled = false;

        // video 'play' event listener
        video.addEventListener('play', function() {
            tick();
        }, false);

        outputImageData = outputCtx.getImageData(0, 0, config.width, config.height);
        outputData = outputImageData.data;
        outputCtx.clearRect(0, 0, config.width, config.height);

    });

    /*
    update video
    */

    function tick() {
        pixel();
        setTimeout(tick, config.interval);
        if( isRecording ){
            if( i < outputData.length * 4 ){
                record();
            } else {
                pause();
                hasEnded = true;
            }
        }
    }

    function pixel() {

        let w = inputCanvas.width / config.resolution;
        let h = inputCanvas.height / config.resolution;
        inputCtx.drawImage(video, 0, 0, w, h);

        let image = inputCtx.getImageData(0, 0, w, h);
        let data = image.data;

        let sum = { r: 0, g: 0, b: 0, i: 0 };

        for( let i = 0; i < data.length; i = i+4 ){
            sum.r += data[i];
            sum.g += data[i+1];
            sum.b += data[i+2];
            sum.i++;
        }

        r = Math.floor( sum.r / sum.i );
        g = Math.floor( sum.g / sum.i );
        b = Math.floor( sum.b / sum.i );

        outputData[i] = r;
        outputData[i+1] = g;
        outputData[i+2] = b;
        outputData[i+3] = 255;

        outputCtx.putImageData(outputImageData, 0, 0);

        image.data.set(data);
        inputCtx.putImageData(image, 0, 0);
    }

    /*
    record
    */

    function getProgress(){
        let progress = Math.round( i / outputData.length * 1000 ) / 10;
        document.title = `Recording ${progress}%`;
    }

    function record(){

        outputData[i] = r;
        outputData[i+1] = g;
        outputData[i+2] = b;
        outputData[i+3] = 255;

        outputCtx.putImageData(outputImageData, 0, 0);

        i += 4;

        getProgress();
    }

    /*
    controls
    */

    function play(){
        isRecording = true;
        hasStarted = true;
        hasEnded = false;
    }

    function handlePlay(){
        buttonText = "Wait...";
        const now = Date.now();
        const startAt = Math.round( ( now + 1000 ) / 2000 ) * 2000;
        const startIn = startAt - now;
        setTimeout(() => {
            play();
        }, startIn);
    }

    function pause(){
        isRecording = false;
        buttonText = 'Resume';
    }

    function reset(){
        buttonText = 'Record';
        hasStarted = false;
        i = 0;
        outputCtx.clearRect(0, 0, config.width, config.height);
    }

    function download(){
        const a = document.createElement('a');
        a.href = outputCanvas.toDataURL('image/png');
        a.download = "received-transmission.png";
        a.click();
        document.body.removeChild(a);
    }

    function handleKeydown( event ){
        if( event.keyCode === 32 ){
            if( isRecording ){
                pause();
            } else {
                handlePlay();        
            }
        }
    }

</script>

<svelte:window on:keydown={handleKeydown}/>

<video class="hidden" bind:this={video} autoplay></video>

<main>

    <section style="background-color:rgb({r},{g},{b});">
        <div>
            <p>
                <span class="minispace">R</span>{r}
                <span class="minispace">G</span>{g}
                <span class="minispace">B</span>{b}
            </p>
        </div>
        <div class="stretch">
            <canvas class="input-video" bind:this={inputCanvas}></canvas>
        </div>
        <div>
            <p style="padding: 0.5em;">&nbsp;</p>
        </div>

    </section>

    <section class="grey">
        <div>
            <p>{i/4} of {config.width * config.height} frames</p>
        </div>

        <div class="stretch">
            <canvas bind:this={outputCanvas} width={config.width*config.resolution} height={config.height*config.resolution}></canvas>
        </div>

        <div>
            {#if isRecording}
                <button on:click={pause}>Stop</button>
            {:else}
                {#if !hasEnded}
                    <button on:click={handlePlay}>{buttonText}</button>
                {/if}
                {#if hasStarted}
                    <button on:click={reset}>Reset</button>
                {/if}
            {/if}
            <!-- {#if hasEnded} -->
                <button on:click={download}>Download</button>
            <!-- {/if} -->
        </div>
    </section>

</main>

<style lang="scss">

    section .stretch {
        display: flex;
        align-items: center;
        justify-content: center;
    }

</style>
