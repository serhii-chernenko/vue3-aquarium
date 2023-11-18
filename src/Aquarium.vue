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
          class="fixed z-10"
          :class="{
            'top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2': !fishes.length,
            'bottom-4 right-4': fishes.length,
          }"
          @click.exact="isAdding = true"
  >
    <span class="block flex items-center justify-center w-24 h-24 p-2 rounded-full bg-cyan-300 shadow-lg hover:bg-white transition-colors add-fish">
      <span class="material-symbols-outlined text-4xl">add</span>
    </span>
  </button>
</template>

<style scoped>
.canvas {
  background: url('/bg.jpg') no-repeat center center;
  background-size: cover;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.add-fish {
  animation: pulse 2s infinite;
}
</style>
