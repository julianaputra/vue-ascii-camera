<script setup lang="ts">
import { onMounted, ref } from 'vue';

const refVideo = ref()
const refHiddenCanvas = ref()
const refOutputCanvas = ref()

const constraints = ref({
  video: {
    width: 510,
    height: 510
  }
})

const setupCamera = async () => {
  const stream = await navigator.mediaDevices.getUserMedia(
    constraints.value
  )
  refVideo.value.srcObject = stream
}

const getRandomCharacter = () => {
  const charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789@#$%&()/\\+<>';
  return charset[Math.floor(Math.random() * charset.length)];
}

const processFrame = () => {
  const { videoWidth: width, videoHeight: height } = refVideo.value
  const fontHeight = 16;

  if (width && height) {
    refHiddenCanvas.value.width = width
    refHiddenCanvas.value.height = height
    refOutputCanvas.value.width = width
    refOutputCanvas.value.height = height

    const hiddenContext = refHiddenCanvas.value.getContext("2d")
    const outputContext = refOutputCanvas.value.getContext("2d")
    hiddenContext.drawImage(refVideo.value, 0, 0, width, height)

    outputContext.textBaseline = 'top'
    outputContext.font = `${fontHeight}px Monospace`;

    const text = outputContext.measureText('@')
    const fontWidth = parseInt(text.width)

    console.log(fontWidth);


    for (let y = 0; y < height; y += fontHeight) {
      for (let x = 0; x < width; x += fontWidth) {
        const frameSection = hiddenContext.getImageData(x, y, fontWidth, fontHeight);
        const { r, g, b } = getAverageRGB(frameSection);


        outputContext.fillStyle = `rgb(${r},${g},${b})`;
        // outputContext.fillRect(x, y, fontWidth, fontHeight);
        outputContext.fillText(getRandomCharacter(), x, y);
      }
    }
  }

  window.requestAnimationFrame(processFrame)
}

const getAverageRGB = (frame) => {
  const length = frame.data.length / 4;

  let r = 0;
  let g = 0;
  let b = 0;

  for (let i = 0; i < length; i++) {
    r += frame.data[i * 4 + 0];
    g += frame.data[i * 4 + 1];
    b += frame.data[i * 4 + 2];
  }

  return {
    r: r / length,
    g: g / length,
    b: b / length,
  };
};

const loadVideo = () => {
  window.requestAnimationFrame(processFrame)
}

onMounted(async () => {
  await setupCamera()
  // setupVideo()
})
</script>

<template>
  <div class="wrapper">
    <video id="video" ref="refVideo" autoplay @loadeddata="loadVideo"></video>
    <canvas id="hiddenCanvas" ref="refHiddenCanvas"></canvas>
    <canvas id="outputCanvas" ref="refOutputCanvas"></canvas>
  </div>
</template>

<style scoped>
#video {
  display: none;
}

#hiddenCanvas {
  display: none;
}

.wrapper {
  flex-wrap: wrap;
  display: flex;
  background-color: black;
}
</style>
