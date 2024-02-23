<template>
  <div class="root">
    <div class="relative">
      <div class="flex flex-row">
        <div class="flex flex-col gap-4 p-4 border-r-2 dark:border-neutral-700 bg-neutral-100 dark:bg-neutral-800 z-50">
          <fwb-button color="default" square shadow="blue" @click="fetchImage()">
            <div class="flex flex-col gap-2 text-left">
              <svg-icon type="mdi" :path="mdiRefresh" />
              <span>New Waifu</span>
            </div>
          </fwb-button>

          <fwb-button color="alternative" square @click="openUrl(imageSrc)">
            <div class="flex flex-col gap-2 text-left">
              <svg-icon type="mdi" :path="mdiImageSearchOutline"></svg-icon>
              <span>See Source</span>
            </div>
          </fwb-button>

          <fwb-button color="alternative" square @click="isShowModal = true">
            <div class="flex flex-col gap-2 text-left">
              <svg-icon type="mdi" :path="mdiAccountDetails"></svg-icon>
              <span>
                by <p class="text-blue-500 font-bold">{{ artistName }}</p>
              </span>
            </div>
          </fwb-button>
        </div>
        <div id="img-div" class="relative w-full h-screen">
          <img class="absolute inset-0 blur-3xl scale-110 m-auto" :src="apiResponseUrl" alt="waifu image" />
          <img class="absolute inset-0 max-h-screen m-auto" :src="apiResponseUrl" alt="waifu image" />
        </div>
      </div>
    </div>
  </div>


  <fwb-modal v-if="isShowModal" @close="closeModal">
    <template #body>
      <div class="flex flex-row justify-between">
        <div class="flex items-center text-lg">
          <span class="dark:text-white">
            Find
            <p class="text-blue-500">{{ artistName }}</p>
            on the internet
          </span>
        </div>
        <div class="flex flex-row gap-4 h-min my-auto">
          <FwbButton color="default">
            <p>Deviant Art</p>
          </FwbButton>
          <FwbButton color="default">
            <p>Pateron</p>
          </FwbButton>
          <FwbButton color="default">
            <p>Pixiv</p>
          </FwbButton>
          <FwbButton color="default">
            <p>Twitter</p>
          </FwbButton>
        </div>
      </div>
    </template>
  </fwb-modal>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { open } from '@tauri-apps/api/shell';
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiRefresh, mdiImageSearchOutline, mdiAccountDetails } from '@mdi/js';
import { FwbButton, FwbModal } from 'flowbite-vue'


const apiUrl = 'https://api.waifu.im/search';
const params = {
  included_tags: 'waifu',
  height: '>=1080',
  width: '>=1920'
};

let apiResponseUrl = ref("");
let imageSrc = ref("");
let artistName = ref("");

let isShowModal = ref(false);

const queryParams = new URLSearchParams(params);
const requestUrl = `${apiUrl}?${queryParams}`;

function fetchImage() {
  fetch(requestUrl)
    .then(response => {
      if (response.ok) {
        response.json().then(data => {
          console.log(data.images[0])
          apiResponseUrl.value = data.images[0].url;
          imageSrc.value = data.images[0].source;
          artistName.value = data.images[0].artist ? data.images[0].artist.name : "Unknown";
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
  try {
    await open(url);
  } catch (error) {
    console.error('An error occurred:', error.message);
  }
}

function closeModal() {
  isShowModal.value = false;
}

onMounted(async () => {
  fetchImage();
});

</script>