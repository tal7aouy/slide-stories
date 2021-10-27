<template>
  <div class="flex flex-col h-screen w-full bg-gray-900">
    <div class="flex flex-row justify-center items-center my-8 h-screen">
      <div
        v-for="(slide, index) in 6"
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
        {{ slide }}
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
    return {
      difference,
      indexSelected,
      selectSlide,
      fetchStories,
      stories,
    }
  },
}
</script>

<style></style>
