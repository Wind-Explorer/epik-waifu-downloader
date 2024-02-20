<template>
  <div style="position: relative; display: flex; flex-direction: row; max-width: 100vw; max-height: 100vh;">
    <div>
      <button @click="fetchImage()">New</button>
      <button @click="openUrl(imageSrc)">Source</button>
    </div>
    <div id="root-div" style="width: 100vw; height: 100vh; display: flex; align-items: center;">
      <img
        style="max-width: calc(100vw - 5vw); max-height: calc(100vh - 5vh); object-fit: contain; border-radius: 10px; margin-top: 10px; margin-bottom: 10px; margin-left: auto; margin-right: auto; border: 2px solid #77777777;"
        :src="apiResponseUrl" alt="waifu image" />
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { open } from '@tauri-apps/api/shell';


const apiUrl = 'https://api.waifu.im/search';
const params = {
  included_tags: 'waifu',
  height: '>=1080',
  width: '>=1920'
};

let apiResponseUrl = ref("");
let dominantColor = ref("");
let imageSrc = ref("");

const queryParams = new URLSearchParams(params);
const requestUrl = `${apiUrl}?${queryParams}`;

function fetchImage() {
  fetch(requestUrl)
    .then(response => {
      if (response.ok) {
        response.json().then(data => {
          apiResponseUrl.value = data.images[0].url;
          dominantColor.value = data.images[0].dominant_color;
          imageSrc.value = data.images[0].source;
          document.getElementById('root-div').style.backgroundColor = dominantColor.value;
        });
      } else {
        throw new Error('Request failed with status code: ' + response.status);
      }
    })
    .catch(error => {
      console.error('An error occurred:', error.message);
    });
}

async function openUrl(url) {
  await open(url);
}

onMounted(async () => {
  // fetchImage();
});

</script>