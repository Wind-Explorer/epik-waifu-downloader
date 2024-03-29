<template>
  <div class="root">
    <div class="relative">
      <div class="flex flex-row">
        <div
          class="flex flex-col justify-between p-4 border-r-2 dark:border-neutral-700 bg-neutral-100 dark:bg-neutral-800 z-50"
        >
          <div class="flex flex-col gap-4">
            <fwb-button
              color="default"
              square
              shadow="blue"
              @click="fetchImage()"
            >
              <div class="flex flex-col gap-2 text-left">
                <svg-icon
                  id="new-img-icn"
                  style="transition: 500ms"
                  type="mdi"
                  :path="mdiRefresh"
                />
                <span>New Waifu</span>
              </div>
            </fwb-button>

            <fwb-button color="alternative" square @click="openUrl(imageSrc)">
              <div class="flex flex-col gap-2 text-left">
                <svg-icon type="mdi" :path="mdiImageSearchOutline"></svg-icon>
                <span>See Source</span>
              </div>
            </fwb-button>
          </div>
          <div
            class="dark:text-white mx-auto p-3"
            @click="
              openUrl(
                'https://www.github.com/wind-explorer/epik-waifu-downloader'
              )
            "
          >
            <svg-icon type="mdi" :path="mdiGithub"></svg-icon>
          </div>
          <!-- artist name   
          <div class="flex flex-col gap-2 text-left dark:text-white bg-neutral-700 p-3 rounded-lg">
            <p class="text-blue-500 font-bold">{{ artistName }}</p>
          </div> -->
        </div>
        <div id="img-div" class="relative w-full h-screen">
          <img
            id="wfimgbg"
            class="absolute inset-0 blur-3xl scale-110 m-auto transition-all duration-1000"
            :src="apiResponseUrl"
            v-show="apiResponseUrl"
            alt="waifu image"
          />
          <img
            id="wfimg"
            class="absolute inset-0 max-h-screen m-auto transition-all duration-1000"
            :src="apiResponseUrl"
            v-show="apiResponseUrl"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import { open } from "@tauri-apps/api/shell";
import SvgIcon from "@jamescoyle/vue-icon";
import { mdiRefresh, mdiImageSearchOutline, mdiGithub } from "@mdi/js";
import { FwbButton } from "flowbite-vue";

import { useToast } from "vue-toast-notification";
import "vue-toast-notification/dist/theme-sugar.css";

const $toast = useToast();

let apiResponseUrl = ref("");
let imageSrc = ref("");
let artistName = ref("");

let img = null;
let imgBg = null;
let newImgIcn = null;

let icnRotateDeg = 0;

const apiUrl = "https://api.waifu.im/search";
const params = {
  included_tags: "waifu",
  height: ">=1080",
  width: ">=1920",
};

const queryParams = new URLSearchParams(params);
const requestUrl = `${apiUrl}?${queryParams}`;

function startChangeImg() {
  img.classList.add("blur-3xl", "opacity-0");
  icnRotateDeg += 360;
  newImgIcn.style.transform = `rotate(${icnRotateDeg}deg)`;

  setTimeout(() => {
    imgBg.classList.add("saturate-0", "opacity-0");
  }, 500);
}

function stopChangeImg() {
  setTimeout(() => {
    imgBg.classList.remove("saturate-0", "opacity-0");
    img.classList.remove("blur-3xl", "opacity-0");
  }, 1500);
}

function fetchImage() {
  startChangeImg();
  fetch(requestUrl)
    .then((response) => {
      if (response.ok) {
        response.json().then((data) => {
          apiResponseUrl.value = data.images[0].url;
          imageSrc.value = data.images[0].source;
          artistName.value = data.images[0].artist
            ? data.images[0].artist.name
            : "Unknown";
        });
      } else {
        $toast.warning("Sorry, no waifu for you. Try again later.");
      }
    })
    .catch((error) => {
      $toast.warning("Fix your internet to see waifu.");
    })
    .finally(() => {
      setTimeout(() => {
        stopChangeImg();
      }, 500);
    });
}

async function openUrl(url) {
  try {
    await open(url);
  } catch (error) {
    $toast.error("Sorry, no source for this one.");
  }
}

function closeModal() {
  isShowModal.value = false;
}

onMounted(async () => {
  img = document.getElementById("wfimg");
  imgBg = document.getElementById("wfimgbg");
  newImgIcn = document.getElementById("new-img-icn");
  $toast.info("Preparing your waifu. Please wait...");
  fetchImage();
});
</script>
