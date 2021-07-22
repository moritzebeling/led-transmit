<script>

    import { onMount } from 'svelte';
    import { config } from '../config.js';

    let isRecording = false;
    let video,
        inputCanvas, inputContext, inputImageData, inputData,
        outputCtx, outputCanvas, outputImageData, outputData;

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

        inputContext = inputCanvas.getContext('2d');
        outputCtx = inputCanvas.getContext('2d');
        inputContext.drawImage(video, 0, 0, width, height);
        inputContext.mozImageSmoothingEnabled = false;
        inputContext.imageSmoothingEnabled = false;

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
        setTimeout(tick, config.intrval);
    }

    function pixel() {

        let w = inputCanvas.width / config.resolution;
        let h = inputCanvas.height / config.resolution;
        inputContext.drawImage(video, 0, 0, w, h);

        let image = inputContext.getImageData(0, 0, w, h);
        let data = image.data;

        let sum = { r: 0, g: 0, b: 0, i: 0 };

        for( let i = 0; i < data.length; i = i+4 ){
            sum.r += data[i];
            sum.g += data[i+1];
            sum.b += data[i+2];
            sum.i++;
        }

        r = sum.r / sum.i;
        g = sum.g / sum.i;
        b = sum.b / sum.i;

        outputData[i] = r;
        outputData[i+1] = g;
        outputData[i+2] = b;
        outputData[i+3] = 255;

        outputCtx.putImageData(outputImageData, 0, 0);

        image.data.set(data);
        inputContext.putImageData(image, 0, 0);
        // inputContext.drawImage(inputCanvas, 0, 0, w, h, 0, 0, config.width, config.height);
        inputContext.drawImage(inputCanvas, 0, 0, w, h, 0, 0, width, height);
    }

    function startRecording(){
        isRecording = true;

    }
    function stopRecording(){
        isRecording = false;
    }

</script>

<video bind:this={video} autoplay></video>

<canvas bind:this={inputCanvas}></canvas>

<div class="display" style="background-color:rgb({r},{g},{b});"></div>

<canvas bind:this={outputCanvas}></canvas>

{#if isRecording}
    <button on:click={stopRecording}>Stop</button>
{:else}
    <button on:click={startRecording}>Start</button>
{/if}

<style lang="scss">

    video, canvas, .display {
        border: 1px solid #ddd;
        display: inline-block;
    }
    .display {
        height: 250px;
        width: 250px;
    }

</style>
