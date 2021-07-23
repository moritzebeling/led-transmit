<script>

    import { onMount } from 'svelte';
    import { config } from '../config.js';

    let video,
        inputCanvas, inputCtx, inputImageData, inputData,
        outputCanvas, outputCtx, outputImageData, outputData;

    let isRecording = false;
    let isStarted = false;

    let r = 0;
    let g = 0;
    let b = 0;

    let i = 0;

    let width = 16;
    let height = 16;
    
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

    function tick() {
        pixel();
        setTimeout(tick, config.interval);
        if( isRecording ){
            record();
        }
    }

    function record(){

        outputData[i] = r;
        outputData[i+1] = g;
        outputData[i+2] = b;
        outputData[i+3] = 255;

        outputCtx.putImageData(outputImageData, 0, 0);

        i += 4;
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

    function startRecording(){
        i = 0;
        isRecording = true;
    }
    function stopRecording(){
        isRecording = false;
    }

</script>

<video bind:this={video} autoplay></video>

<section class="full">

    <div class="panel input" style="background-color:rgb({r},{g},{b});">
        <div>
            <p>R{r} G{g} B{b}</p>
        </div>
        <div class="main">
            <canvas class="input-video" bind:this={inputCanvas}></canvas>
        </div>
        <div>
            <p></p>
        </div>

    </div>

    <div class="panel output">
        <div>
            <p>{i/4} of {config.width * config.height} frames</p>
        </div>

        <div class="main">
            <canvas bind:this={outputCanvas} width={config.width*config.resolution} height={config.height*config.resolution}></canvas>
        </div>

        <div>
            {#if isRecording}
                <button on:click={stopRecording}>Stop</button>
            {:else}
                <button on:click={startRecording}>Start</button>
            {/if}
        </div>
    </div>

</section>

<style lang="scss">

    video {
        position: fixed;
        top: -9999px;
        left: -9999px;
    }
    canvas {
        background-color: #000;
    }
    section {
    }
    .panel {
        flex: 1;
        display: flex;
        flex-direction: column;
    }
    .panel > div {
        padding: 1rem;
    }
    .panel.input {
        justify-content: space-between;
    }
    .panel .main {
        flex: 1;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .panel.output {
        background-color: #222;
    }

</style>
