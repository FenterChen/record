<script setup lang="ts">
import { ref } from "vue";
const videoMediaRecorder = ref<any>(null);
const stream = ref<any>(null);
const toggleValue = ref<boolean>(false);
const loop = ref<any>(false);
const type = ref<any>();


async function Record() {
  if (videoMediaRecorder.value) return
  const device = await navigator.mediaDevices.enumerateDevices()
  console.log(device)

  const constraints = {
    width: { ideal: 1920 },
    height: { ideal: 1080 },
    frameRate: { ideal: 24 },
    // deviceId: "dae47848fb74c91d0059d734409af9ae39ad1511b24e28ebb50076009a64f289"
    deviceId: "cd336532e9c95611156811d0853a7c12195552a59e68a19f3034cb39178f4333"
  };
  // let stream  = await navigator.mediaDevices.getUserMedia({ video: constraints });
  // stream.getVideoTracks()[0].enabled
  stream.value = await navigator.mediaDevices.getUserMedia({ video: constraints });
  const elem = document.createElement("video");
  elem.srcObject = stream.value;
  elem.autoplay = true;
  elem.width = 1280;
  elem.height = 720;
  document.getElementById('media')?.appendChild(elem);
  let bit: number;
  if (type.value == "Repair_") {
    bit = 5000 * 1000;
  } else {
    bit = 4000 * 1000;
  };
  const options = {
    videoBitsPerSecond: bit,
    mimeType: 'video/webm',
  }
  const mediaRecorder = new MediaRecorder(stream.value, options)
  videoMediaRecorder.value = mediaRecorder;
  videoMediaRecorder.value.addEventListener('dataavailable', (e: { data: BlobPart; }) => {
    const blob = new Blob([e.data], { type: 'video/mp4' })
    const downloadLink = document.createElement('a')
    downloadLink.href = window.URL.createObjectURL(blob)
    let dt = new Date();
    downloadLink.download = `${type.value}${dt}`
    downloadLink.click()
  })
  videoMediaRecorder.value.start();
  loop.value = setInterval(VideoEndDownload, 1000 * 60 * 10);
}

function EndRecording() {
  if (toggleValue.value == false) {
    stream.value.getVideoTracks()[0].enabled = false;
    toggleValue.value = true;
    console.log(videoMediaRecorder.value)
    videoMediaRecorder.value.stop();
    clearInterval(loop.value);
    location.reload();
  } else if (toggleValue.value == true) {
    stream.value.getVideoTracks()[0].enabled = true;
    toggleValue.value = false;
    videoMediaRecorder.value.start();
    loop.value = setInterval(VideoEndDownload, 1000 * 60 * 10);
    location.reload();
  }
}
function VideoEndDownload() {
  if (!videoMediaRecorder.value) return
  videoMediaRecorder.value.stop();
  videoMediaRecorder.value.start();
}

</script>

<template>
  <div class="my-2 mx-3 py-1 px-2">
    <h1 class="text-lg font-semibold">Record Media</h1>
    <div class="flex items-center">
      <h2 class="mr-2">Plz Choose Record Type: </h2>
      <select class="mr-2" v-model="type">
        <option value="Repair_">Repair</option>
        <option value="OK_">OK</option>
        <option value="蝦皮_">蝦皮</option>
        <option value="711_">711</option>
        <option value="Family_">Family</option>
      </select>
      <button v-if="type"
        class="m-btn mr-2 bg-slate-900 hover:bg-slate-700 focus:outline-none focus:ring-2 focus:ring-slate-400 focus:ring-offset-2 focus:ring-offset-slate-50 text-white font-semibold h-12 px-6 rounded-lg w-full flex items-center justify-center sm:w-auto dark:bg-sky-500 dark:highlight-white/20 dark:hover:bg-sky-400"
        @click="Record">Open Camera Video</button>
      <button v-if="type"
        class="m-btn mr-2 bg-red-900 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-400 focus:ring-offset-2 focus:ring-offset-red-50 text-white font-semibold h-12 px-6 rounded-lg w-full flex items-center justify-center sm:w-auto dark:bg-sky-500 dark:highlight-white/20 dark:hover:bg-sky-400"
        @click="EndRecording">End Recording</button>
    </div>
    <div id="media"></div>
  </div>
</template>
