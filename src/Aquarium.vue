<script setup>
import { shallowRef, shallowReactive, provide } from "vue";
import FishForm from "./FishForm.vue";
import Fish from "./Fish.vue";

const fishes = shallowReactive([]);
const isAdding = shallowRef(false);

provide('isFishFormOpened', isAdding);
provide('fishes', fishes);

const deleteFish = id => {
  fishes.splice(fishes.findIndex(fish => fish.id === id), 1);
};
</script>

<template>
  <FishForm @closeFishForm="isAdding = false"/>
  <div class="relative h-screen w-screen overflow-hidden canvas">
    <Fish v-for="fish in fishes"
          :key="fish.id"
          v-bind="fish"
          @deleteFish="deleteFish"
    />
  </div>
  <button type="button"
          title="Add a fish"
          class="fixed z-10 bottom-4 right-4 flex items-center justify-center w-24 h-24 p-2 rounded-full bg-white shadow-lg hover:bg-cyan-300"
          @click.exact="isAdding = true"
  >
    <span class="material-symbols-outlined text-4xl">add</span>
  </button>
</template>

<style scoped>
.canvas {
  background: url('/bg.jpg') no-repeat center center;
  background-size: cover;
}
</style>
