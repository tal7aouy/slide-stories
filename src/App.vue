<template>
  <div class="flex flex-col h-screen w-full bg-gray-900">
    <div class="flex flex-row justify-center items-center my-8 h-screen">
      <div
        v-for="(story, index) in stories"
        :key="index"
        class="
          absolute
          mx-2
          rounded-lg
          bg-red-400
          transition-transform
          duration-200
          ease-in
        "
        style="width: 350px; height: 580px"
        :style="
          index == indexSelected
            ? `transform: translate(${380 * (index + difference)}px) scale(0.9)`
            : `transform: translate(${
                280 * (index + difference)
              }px) scale(0.4);cursor:pointer;`
        "
        @click="index != indexSelected ? selectSlide(index) : ''"
      >
        <div class="bg-cover bg-no-repeat h-full rounded-lg">
          <div class="h-full">
            <img
              :src="
                index == indexSelected
                  ? story.images[key].url
                  : story.images[0].url
              "
              class="h-full w-full rounded-lg"
            />
          </div>
          <div class="w-full pt-4 absolute top-0" v-if="index == indexSelected">
            <div class="w-11/12 flex m-auto">
              <div
                class="w-full rounded-lg mr-2 relative h-auto"
                v-for="(elm, index) in story.images.length"
                :key="index"
              >
                <!-- length slide -->
                <div
                  class="w-full rounded-lg"
                  style="
                    height: 4px;
                    background-color: rgba(255, 255, 255, 0.35);
                  "
                ></div>
                <!-- end length -->
                <div
                  class="w-full rounded-lg"
                  style="height: 4px; background-color: white"
                ></div>
              </div>
            </div>
            <div class="flex w-11/12 mt-4 m-auto">
              <div class="flex justify-start items-center w-1/2">
                <div style="width: 35px; height: 35px">
                  <img :src="story.picture" class="rounded-full" />
                </div>
                <div class="ml-4">
                  <p class="text-sm text-white font-semibold">
                    {{ story.username }}
                  </p>
                </div>
              </div>
              <div class="flex justify-end items-center w-1/2">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-5 w-5 cursor-pointer"
                  viewBox="0 0 20 20"
                  fill="#ffffff"
                >
                  <path
                    fill-rule="evenodd"
                    d="M10.293 15.707a1 1 0 010-1.414L14.586 10l-4.293-4.293a1 1 0 111.414-1.414l5 5a1 1 0 010 1.414l-5 5a1 1 0 01-1.414 0z"
                    clip-rule="evenodd"
                  />
                  <path
                    fill-rule="evenodd"
                    d="M4.293 15.707a1 1 0 010-1.414L8.586 10 4.293 5.707a1 1 0 011.414-1.414l5 5a1 1 0 010 1.414l-5 5a1 1 0 01-1.414 0z"
                    clip-rule="evenodd"
                  />
                </svg>
              </div>
            </div>
          </div>
          <div v-else>
            <div class="absolute top-1/2 left-1/2 z-50">
              <div class="flex flex-col items-center">
                <div
                  style="
                    width: 50px;
                    height: 50px;
                    transform: translate(-50%, -50%) scale(2.5);
                  "
                  class="
                    rounded-full
                    border-2 border-indigo-400
                    transition-transform
                    duration-100
                  "
                >
                  <img :src="story.picture" class="rounded-full" />
                </div>
                <div class="mt-2">
                  <p class="text-sm text-white font-semibold">
                    {{ story.username }}
                  </p>
                </div>
              </div>
            </div>
            <div
              class="absolute inset-0 rounded-lg z-10"
              style="
                background: -webkit-gradient(
                  linear,
                  left top,
                  left bottom,
                  from(rgba(38, 38, 38, 0.6)),
                  to(rgba(38, 38, 38, 0))
                );
              "
            ></div>
          </div>
        </div>
        <div v-if="index == indexSelected" class="absolute top-1/2 -left-11">
          <i
            @click="prev(index)"
            class="
              fas
              fa-chevron-circle-left
              text-gray-500
              cursor-pointer
              hover:text-gray-300
              transition-colors
              duration-150
              ease-linear
            "
          ></i>
        </div>
        <div v-if="index == indexSelected" class="absolute top-1/2 -right-11">
          <i
            @click="next(index)"
            class="
              fas
              fa-chevron-circle-right
              text-gray-500
              cursor-pointer
              hover:text-gray-300
              transition-colors
              duration-150
              ease-linear
            "
          ></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue'
import { axios } from './axios'
export default {
  name: 'App',
  setup() {
    const indexSelected = ref(0)
    const difference = ref(0)
    const stories = ref([])
    const key = ref(0)
    const selectSlide = (index) => {
      difference.value += indexSelected.value - index
      indexSelected.value = index
    }
    const fetchStories = async () => {
      const response = await axios
        .get('/items')
        .catch((err) => console.log(err))
      if (response && response.data) {
        stories.value = response.data
      }
    }
    onMounted(() => {
      fetchStories()
    })
    const next = (index) => {
      if (
        indexSelected.value >= stories.value.length - 1 &&
        key.value >= stories.value[indexSelected.value].images.length - 1
      ) {
        // Without delay
        setTimeout(() => {
          difference.value = 0
          indexSelected.value = 0
          key.value = 0
        })
      } else if (
        key.value >=
        stories.value[indexSelected.value].images.length - 1
      ) {
        // Without delay
        setTimeout(() => {
          difference.value += index - (index + 1)
          indexSelected.value++
          key.value = 0
        })
      } else {
        key.value++
      }
    }
    const prev = () => {
      key.value--
    }
    return {
      difference,
      indexSelected,
      selectSlide,
      fetchStories,
      stories,
      next,
      prev,
      key,
    }
  },
}
</script>

<style>
.fas {
  font-size: 30px;
}
</style>
