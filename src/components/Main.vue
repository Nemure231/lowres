<script setup>
import { ref, onUnmounted, computed } from 'vue';
import { saveAs } from 'file-saver';
import { Cropper } from 'vue-advanced-cropper'
import { compressAccurately } from 'image-conversion';
import 'vue-advanced-cropper/dist/theme.compact.css';
import 'vue-advanced-cropper/dist/style.css';

const example = ref([
  {
    id: 1,
    name: 'Bocchi',
    url: 'img/example/1.jpg'
  },
  {
    id: 2,
    name: 'Fumo Cirno',
    url: 'img/example/2.jpg'
  },
  {
    id: 3,
    name: 'Honda Mio',
    url: 'img/example/3.jpg'
  },
  {
    id: 4,
    name: 'Asakusa',
    url: 'img/example/4.jpg'
  },
  {
    id: 5,
    name: 'Nadeshiko',
    url: 'img/example/5.jpg'
  },
  {
    id: 6,
    name: 'Akari',
    url: 'img/example/6.jpg'
  },
  {
    id: 7,
    name: 'Shamiko',
    url: 'img/example/7.jpg'
  },
  {
    id: 8,
    name: 'Japanese Goblin',
    url: 'img/example/8.jpg'
  }
])




const rangeQuality = ref(25)

const scaleQuality = ref(1)

const urlImg = ref('')
const coor = ref({
  width: 0,
  height: 0,
  left: 0,
  top: 0,
})


const image = ref({
  src: null,
  type: null
})

const selectExample = ref('')
const cropedImg = ref({
  src: null,
  size: null
})

const blobImg = ref(null)
const compressedImg = ref(null)




// Get default location
let getLocation = () => {
  return window.location.href;
}


// Get mime type
let reduceQualityimeType = (file, fallback = null) => {
  const byteArray = (new Uint8Array(file)).subarray(0, 4);
  let header = '';
  for (let i = 0; i < byteArray.length; i++) {
    header += byteArray[i].toString(16);
  }
  switch (header) {
    case "89504e47":
      return "image/png";
    case "47494638":
      return "image/gif";
    case "ffd8ffe0":
    case "ffd8ffe1":
    case "ffd8ffe2":
    case "ffd8ffe3":
    case "ffd8ffe8":
      return "image/jpeg";
    default:
      return fallback;
  }
}
const file = ref(null)
// choose link or local image
let upload = async () => {
  // If url exist in input
  if (urlImg.value) {
    // Get the result from link validation
    const checkUrlType = checkUrlImg(urlImg.value);
    // And check the result
    if (checkUrlType === true) {

      // Make a fetch to check if the website have CORS policy or not
      await fetch(urlImg.value, {
        method: 'GET',
        headers: {
        },
      })
        .then(response => response.blob())
        .then(data => {
          image.value.src = urlImg.value
        })
        .catch((error) => {
          alert('The url of this image is protected or blocked by CORS policy, if you want this image just download it and choose again.')
        });

    } else {
      // Else return the alert message
      alert('The url only accept https, and with ,jpg, .jpeg, and .png extension in the end of url!')
    }
    // Else people select form their local storage
  } else {
    file.value.click()
  }
}

let loadImage = (event) => {
  const { files } = event.target;
  // Reference to the DOM input element
  // Ensure that you have a file before attempting to read it
  if (files && files[0]) {
    const fileType = files[0]['type'];
    const fileSize = files[0]['size'];
    const validImageTypes = ['image/gif', 'image/jpeg', 'image/png'];
    // TYPE IMAGE VALIDATION
    if (!validImageTypes.includes(fileType)) {
      alert('Please, choose the right format image! The avaiable format is JPG, JPEG, and PNG!');
    }
    //SIZE IMAGE VALIDATION
    else if (fileSize >= 1045301 * 2) {
      alert('The maximum size image to upload is 2MB, please reduce your image size before upload again!');
    } else {

      if (image.value.src) {
        URL.revokeObjectURL(image.value.src)
      }
      const blob = URL.createObjectURL(files[0]);

      const reader = new FileReader();
      reader.onload = (e) => {
        image.value = {
          src: blob,
          type: reduceQualityimeType(e.target.result, files[0].type),
        };
      };
      reader.readAsArrayBuffer(files[0]);
    }

  }
}

// base64 to blob async function
let b64toBlob = async (url) => {
  await fetch(url)
    .then(
      res => res.blob()
    )
    .then(data => {
      blobImg.value = data;
    });
}

// Check if the url have https or have extension
let checkUrlImg = async (url) => {
  if (url.match(/^https:?:\/\/.+\/.+$/) && url.match(/\.(jpeg|jpg|png)$/) !== null) {
    return true
  } else {
    return false
  }
}

const cropp = ref(null)
// Save image function
let saveImg = () => {
  saveAs(compressedImg.value, `${Date.now()}.jpg`);
}
// Crop image function
let getCrop = () => {
  // Reset compressed image and range quality if there's a new image
  compressedImg.value = null;
  rangeQuality.value = 25;
  scaleQuality.value = 1;

  const { coordinates, canvas, } = cropp.value.getResult();

  coor.value = coordinates;
  const files = canvas.toDataURL();
  cropedImg.value.src = files;
  b64toBlob(files)
}
// Reduce quality function
let reduceQuality = () => {
  compressAccurately(blobImg.value, {
    size: rangeQuality.value,
    scale: scaleQuality.value,
  }).then(res => {
    const blobUrl = URL.createObjectURL(res)
    compressedImg.value = blobUrl
  })

}


onUnmounted(() => {
  if (image.value.src) {
    URL.revokeObjectURL(image.value.src)
  }
})


// Some conditions 
const isCroped = computed(() => cropedImg.value.src ? 'block' : 'hidden')
const isUploaded = computed(() => image.value.src ? 'block' : 'hidden')
const isBg = computed(() => image.value.src ? 'h-auto' : 'bg-teal-50  md:h-72 2xl:h-[25rem] lg:h-[20rem] sm:h-64 h-52')
const isMargin = computed(() => image.value.src ? 'mb-0' : 'mb-32')
const isHeight = computed(() => cropedImg.value.src ? '' : 'bg-teal-50')


const isEffect = computed(() => compressedImg.value ? compressedImg.value : cropedImg.value.src)

const isLowresMember = computed(() => {
  if (rangeQuality.value >= 20 && rangeQuality.value <= 25) {
    return `Are you really lowres member? 🤨`
  }

  if (rangeQuality.value > 25) {
    return `Wait, you are not lowres member 😠`
  }

  if (rangeQuality.value < 20) {
    return `You are lowres member, show this to your homies 😎`;
  }
})

</script>
<template>
  <main class="w-full flex flex-wrap flex-col justify-center items-center">
    <section class="flex-1 mt-12 mb-8 relative w-full">
      <div class="flex justify-center flex-wrap">
        <div class="relative flex-1 2xl:max-w-7xl lg:max-w-6xl md:max-w-3xl sm:max-w-xl max-w-max lg:mx-0 md:mx-0 mx-4">

          <div
            class="flex lg:gap-12 md:gap-12 sm:gap-14 gap-4 justify-center lg:flex-row md:flex-row flex-col flex-wrap lg:items-start md:items-start items-center md:mx-3 mx-0">

            <div :class="[isBg, isMargin]"
              class="lg:flex-1 lg:mb-0 md:mb-0  md:flex-1 flex-none  rounded-xl  2xl:w-[20rem]  lg:w-[18rem] md:w-72 w-full">
              <div
                class="flex h-full flex-col items-center justify-center lg:w-full md:w-full sm:w-full w-[15rem] mx-auto">
                <cropper ref="cropp" :src="image.src">
                </cropper>
              </div>
              <div
                class="lg:mt-3 md:mt-3 sm:mt-3 mt-6 lg:text-base  flex-wrap md:text-base sm:text-sm text-xs flex flex-row gap-3 lg:justify-start md:justify-start justify-center items-center">
                <select @change="image.src = selectExample" v-model="selectExample" name="example"
                  class="text-gray-500 border lg:w-1/3 w-full border-gray-400 rounded-md px-3 lg:py-1.5 md:py-1.5 py-2 focus:outline-none">
                  <option selected value="">Select example</option>
                  <option v-text="ex.name" v-for="ex in example" :key="ex.id" :value="`${getLocation()}${ex.url}`">

                  </option>
                </select>

                <div class="relative lg:w-1/2 w-full">
                  <div class="flex items-center">
                    <input v-model="urlImg"
                      class="relative border w-full border-gray-400 rounded-md pl-3 pr-28 lg:py-1.5 md:py-1.5 py-2 focus:outline-none"
                      type="url" placeholder="Link or local img ...">

                    <button
                      class="absolute  right-1 font-semibold lg:px-6  lg:py-1 md:px-6 md:py-1 px-3 py-1 text-white  rounded-md bg-[#0AA1DD]"
                      @click="upload()">
                      <input class="hidden" type="file" ref="file" id="img-target" @change="loadImage($event)"
                        accept="image/*">
                      Choose
                    </button>

                  </div>
                </div>

                <button @click="getCrop" :class="isUploaded"
                  class="lg:w-auto w-full font-semibold lg:px-6 lg:py-1.5 md:px-6 md:py-1.5 px-4 py-1.5 text-white text-sm rounded-md bg-[#0AA1DD]">
                  Crop
                </button>
              </div>

            </div>

            <div :class="isHeight"
              class="lg:w-[18rem] md:w-64 w-60 mb-6 lg:h-[18rem] md:h-64 h-60  z-10 rounded-xl flex-none text-center">
              <div :class="isCroped" class="flex flex-col gap-3 justify-center items-center mt-4">
                <img class="w-auto h-auto object-cover" :src="isEffect" alt="">
                <span class="text-sm" v-text="isLowresMember"></span>
                <label for="" class="text-xs text-left w-full font-semibold -mb-2">Quality</label>
                <input type="range" class="w-full" v-model="rangeQuality" @change="reduceQuality" min="1" max="50">
                <label for="" class="text-xs text-left w-full font-semibold -mb-2">Scale</label>
                <input type="range" class="w-full" v-model="scaleQuality" @change="reduceQuality" min="0.1" max="1"
                  step="0.01">
                <button
                  class=" font-semibold px-6 py-1.5 lg:mb-0 md:mb-0 mb-14 lg:text-base md:text-base sm:text-sm text-xs text-white  rounded-md bg-[#0AA1DD]"
                  @click="saveImg()">
                  Download
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>