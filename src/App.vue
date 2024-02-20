<template>
  <div
    style="position: relative; display: flex; flex-direction: row; max-width: 100vw; max-height: 100vh; border: 1px solid red;">
    <div>
      <button @click="fetchImage()">New</button>
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

const apiUrl = 'https://api.waifu.im/search';
const params = {
  included_tags: 'waifu',
  height: '>=1080',
  width: '>=1920'
};

let apiResponseUrl = ref("");
let dominant_color = ref("");

const queryParams = new URLSearchParams(params);
const requestUrl = `${apiUrl}?${queryParams}`;

function fetchImage() {
  fetch(requestUrl)
    .then(response => {
      if (response.ok) {
        response.json().then(data => {
          apiResponseUrl.value = data.images[0].url;
          dominant_color.value = data.images[0].dominant_color;
          document.getElementById('root-div').style.backgroundColor = dominant_color.value;
        });
      } else {
        throw new Error('Request failed with status code: ' + response.status);
      }
    })
    .catch(error => {
      console.error('An error occurred:', error.message);
    });
}

onMounted(async () => {
  fetchImage();
});

</script>