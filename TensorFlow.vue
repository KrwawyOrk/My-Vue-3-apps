<template>
  <div class="flex w-full">
    <div class="m-auto media-size-properties">
      <div id="tensor-flow-container" class="rounded-2xl border shadow">
        <div class="flex flex-col p-2">
          <!-- Hello Tensorflow.js section -->
          <section id="hello-tensorflow">
            <div>
              <h1>Hello,<span></span></h1>
              <h1>My name is<span></span></h1>
              <h1>Tensor Flow!<span></span></h1>
            </div>
          </section>
          <!-- End hello Tensorflow.js section -->

          <!-- Buttons and image url section -->
          <section class="flex flex-col justify-center">
            <div class="flex flex-col justify-center">
              <input
                v-model="state.imageLinkInput"
                type="text"
                placeholder="Paste and image url here!"
                class="p-2 my-2 rounded-md border shadow"
              />
              <button
                @click="setImage"
                class="p-2 my-1 bg-gradient-to-t from-gray-800 to-gray-700 text-white px-3 rounded-md hover:from-gray-700"
              >
                Set image
              </button>
            </div>
            <button
              @click="changeImage"
              class="p-2 my-1 bg-gradient-to-t from-gray-800 to-gray-700 text-white px-3 rounded-md hover:from-gray-700"
            >
              Get random image!
            </button>
            <button
              @click="detectObject"
              :disabled="state.isDetecting ? true : false"
              class="p-2 my-1 bg-gradient-to-t from-red-900 to-red-800 text-white px-3 rounded-md hover:from-red-800"
            >
              <span v-if="!state.isDetecting">Detect object</span>
              <span v-else>I am detecting object...</span>
            </button>
            <div class="h-14 mt-5">
              <h1
                v-if="state.detectionResult"
                class="p-2 text-xl text-center bg-gray-700 text-white rounded-md fade-in"
              >
                {{ state.detectionResult }}
              </h1>
            </div>
          </section>
          <!-- End Buttons and image url section -->

          <!-- Image to detect section -->
          <section class="flex justify-center">
            <img
              ref="imageToDetect"
              crossorigin="anonymous"
              class="rounded-md"
              style="width: 90%"
            />
          </section>
          <!-- End Image to detect section -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";

import * as cocoSsd from "@tensorflow-models/coco-ssd";
import { onMounted, ref, reactive, nextTick } from "vue";

export default {
  setup() {
    const imageToDetect = ref("");

    const state = reactive({
      isDetecting: false,
      imageLinkInput: "",
      detectionResult: "",
    });

    onMounted(() => {
      imageToDetect.value.src =
        "https://images.unsplash.com/photo-1561157437-415893bd7149";
    });

    async function detectObject() {
      clearInputAndResult();

      state.isDetecting = true;
      const model = await cocoSsd.load();

      model
        .detect(imageToDetect.value)
        .then((predictions) => {
          console.log(predictions[0].class);

          const detectedObjectName = predictions[0].class;
          state.detectionResult = `You have detected ${detectedObjectName}`;
        })
        .catch((error) => {
          console.log(error);
          state.detectionResult = "Sorry! I can't detect the picture :(";
        })
        .finally(() => {
          state.isDetecting = false;
        });
    }

    function changeImage() {
      clearInputAndResult();

      imageToDetect.value.src =
        "https://images.unsplash.com/photo-1489824904134-891ab64532f1";
    }

    function setImage() {
      if (state.imageLinkInput) {
        imageToDetect.value.src = state.imageLinkInput;

        clearInputAndResult();
      } else {
        clearInputAndResult();

        nextTick(() => {
          state.detectionResult = "You need an image link! :)";
        });
      }
    }

    function clearInputAndResult() {
      state.imageLinkInput = "";
      state.detectionResult = "";
    }

    return {
      cocoSsd,

      detectObject,
      changeImage,
      setImage,

      imageToDetect,
      state,
    };
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Montserrat:wght@900&display=swap");
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  font-family: "Montserrat", sans-serif;
}

.media-size-properties {
  width: 30%;
}

@media screen and (max-width: 1024px) {
  .media-size-properties {
    width: 50%;
  }
}

@media screen and (max-width: 600px) {
  .media-size-properties {
    width: 90%;
  }
}

.fade-in {
  animation: fadeIn ease 0.5s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    width: 50%;
  }

  to {
    opacity: 1;
    width: 100%;
  }
}

#tensor-flow-container {
background-image: url('https://images.unsplash.com/photo-1555255707-c07966088b7b?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1490&q=80');
  background-size: cover;
  background-position: relative;
  background-position: top center;
}

#hello-tensorflow {
  display: flex;
  flex-direction: column;
  align-items: center;

  margin-top: 30px;

  background-color: black;
  
  border-radius: 10px;
  border: 2px solid;
  padding: 50px;

  animation: container_effect 1s ease forwards;
  animation-delay: 4s;
}

#hello-tensorflow h1 {
  display: block;
  width: fit-content;
  position: relative;
  font-size: 20px;
  font-weight: bolder;
  color: transparent;
  animation: text_reveal .5s ease forwards;
  animation-delay: 1s;
}

#hello-tensorflow h1:nth-child(1) {
  animation-delay: 1s;
}

#hello-tensorflow h1:nth-child(2) {
  animation-delay: 2s;
}

#hello-tensorflow h1:nth-child(3) {
  animation-delay: 3s;
}

#hello-tensorflow h1 span {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 0;
  background-color: crimson;

  animation: text_reveal_box 1s ease forwards;
  animation-delay: .3s;
}

#hello-tensorflow h1:nth-child(1) span {
  animation-delay: 0.5s;
}

#hello-tensorflow h1:nth-child(2) span {
  animation-delay: 1.5s;
}

#hello-tensorflow h1:nth-child(3) span {
  animation-delay: 2.5s;
}

/* Keyframes */

@keyframes text_reveal {
  100% {
    color: white;
  }
}

@keyframes text_reveal_box {
  0% {
    width: 100%;
    left: 0%;
  }

  100% {
    width: 0%;
    left: 100%;
  }
}

@keyframes container_effect {
  0% {
    opacity: 1;
  }

  100% {
    opacity: 0;
  }
}


/* End Keyframes */
</style>