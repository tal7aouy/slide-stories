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
                  class="absolute w-full rounded-lg"
                  style="
                    height: 4px;
                    background-color: rgba(255, 255, 255, 0.35);
                  "
                ></div>
                <!-- end length -->
                <div
                  class="absolute w-full rounded-lg"
                  style="height: 4px; background-color: white"
                  :style="
                    index == key
                      ? `width: ${percent}%`
                      : key > index
                      ? `width:100%`
                      : `width:0%`
                  "
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
                  class="h-6 w-6 cursor-pointer"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke="#fff"
                  @click="isPaused ? playStory() : pauseStory()"
                >
                  <path
                    v-if="!isPaused"
                    d="M9 6a1 1 0 0 1 1 1v10a1 1 0 1 1-2 0V7a1 1 0 0 1 1-1zm6 0a1 1 0 0 1 1 1v10a1 1 0 1 1-2 0V7a1 1 0 0 1 1-1z"
                    fill="#fff"
                  />
                  <path
                    v-else
                    d="M6 6.741c0-1.544 1.674-2.505 3.008-1.728l9.015 5.26c1.323.771 1.323 2.683 0 3.455l-9.015 5.258C7.674 19.764 6 18.803 6 17.26V6.741zM17.015 12L8 6.741V17.26L17.015 12z"
                    fill="#fff"
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
import { data } from './stories'
export default {
  name: 'App',
  setup() {
    // declaration side
    const indexSelected = ref(0)
    const difference = ref(0)
    const stories = ref([])
    const key = ref(0)
    // percent story
    const percent = ref(0)
    const timer = ref(0)
    const progress = ref(0)
    const duration = ref(5000)
    const interval = ref(0)
    const isPaused = ref(false)
    const newDur = ref(0) // duration
    const pausePer = ref(0) // pausePercent
    // ----end declaration
    const selectSlide = (index) => {
      difference.value += indexSelected.value - index
      indexSelected.value = index
      key.value = 0
      reset()
    }
    const fetchStories = () => {
      stories.value = data
      play()
    }

    // next story
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
      reset()
    }
    // previous story
    const prev = (index) => {
      if (indexSelected.value <= 0 && key.value <= 0) {
        key.value = 0
      } else if (key.value <= 0) {
        // Without delay
        setTimeout(() => {
          difference.value += index - (index - 1)
          indexSelected.value--
          key.value = 0
        })
      } else {
        key.value--
      }
      reset()
    }
    const autoPlay = () => {
      if (
        indexSelected.value >= stories.value.length - 1 &&
        key.value >= stories.value[indexSelected.value].images.length - 1
      ) {
        difference.value = 0
        indexSelected.value = 0
        key.value = 0
      } else if (
        key.value >=
        stories.value[indexSelected.value].images.length - 1
      ) {
        difference.value += indexSelected.value - (indexSelected.value + 1)
        indexSelected.value++
        key.value = 0
      } else {
        key.value++
      }
      reset()
    }
    const play = () => {
      timer.value = new Date().getTime()
      progress.value = setInterval(() => {
        // forward
        let time = new Date().getTime()
        if (newDur.value > 0) {
          percent.value =
            pausePer.value +
            Math.floor((100 * (time - timer.value)) / duration.value)
        } else {
          percent.value = Math.floor(
            (100 * (time - timer.value)) / duration.value
          )
        }
      }, duration.value / 100)
      if (newDur.value > 0) {
        interval.value = setInterval(autoPlay, newDur.value)
      } else {
        interval.value = setInterval(autoPlay, duration.value)
      }
    }
    // reset play
    const reset = () => {
      percent.value = 0
      clearInterval(interval.value)
      clearInterval(progress.value)
      play()
    }
    const pauseStory = () => {
      isPaused.value = true
      pausePer.value = percent.value
      clearInterval(progress.value)
      clearInterval(interval.value)
      // calcul new duration
      newDur.value = duration.value - (pausePer.value * duration.value) / 100
    }
    const playStory = () => {
      isPaused.value = false
      play()
    }
    // OnMounted hook
    onMounted(() => {
      fetchStories()
    })
    return {
      difference,
      indexSelected,
      selectSlide,
      fetchStories,
      stories,
      next,
      prev,
      key,
      percent,
      play,
      timer,
      progress,
      duration,
      reset,
      interval,
      autoPlay,
      isPaused,
      pauseStory,
      playStory,
      newDur,
      pausePer,
    }
  },
}
</script>

<style>
/* import fonts */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500&display=swap');
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font-family: 'Poppins', sans-serif;
}
.fas {
  font-size: 30px;
}
</style>
