<script setup>
import { onMounted, onUnmounted, inject, shallowRef, shallowReactive } from "vue";

const emit = defineEmits(['closeFishForm']);
const isFishFormOpened = inject('isFishFormOpened');
const fishes = inject('fishes');

onMounted(() => window.addEventListener('keydown', keydownListeners));
onUnmounted(() => window.removeEventListener('keydown', keydownListeners));

const keydownListeners = event => {
  if (!isFishFormOpened.value) {
    return false;
  }

  if (event.key === 'Escape') {
    closeModal();
  }
}

const closeModal = () => {
  emit('closeFishForm');
  name.value = '';
};

// See names in public .png images
const types = shallowReactive(['golden-purple-fish', 'goldfish', 'guppie', 'tropical-fish', 'tuna']);
const randomNames = ['Thor', 'Locky', 'Iron Man', 'Hulk', 'Cap', 'Marvel', 'Dr. Strange', 'Vision', 'Wanda', 'Spider Man'];
const name = shallowRef('');
const type = shallowRef(types[0]);

const addFish = () => {
  if (!name.value) {
    name.value = getRandomName();
  }

  fishes.push({
    id: window.crypto.randomUUID(),
    name: name.value,
    type: type.value
  });
  closeModal();
};

const getRandomName = () => randomNames[Math.floor(Math.random() * randomNames.length)];
</script>
<template>
  <transition name="overlay">
    <div
        @click="closeModal"
        v-if="isFishFormOpened"
        tabindex="0"
        title="Close"
        class="fixed z-40 inset-0 bg-black bg-opacity-50 cursor-pointer"
    ></div>
  </transition>
  <transition name="modal" type="animation" :duration="500">
    <template v-if="isFishFormOpened">
      <div class="fixed z-50 top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 inset-0 flex items-center justify-center">
        <div class="bg-white rounded-lg p-8">
          <h2 class="mb-4 text-2xl text-center">Add a new fish</h2>
          <form @submit.prevent="addFish">
            <div class="mb-4">
              <label for="type" class="block mb-2">Type</label>
              <ul class="grid grid-cols-5 gap-4 items-center">
                <li v-for="(fish, index) in types" :key="index">
                  <label class="cursor-pointer">
                    <input type="radio" name="type" v-model="type" :value="fish" class="appearance-none">
                    <img :src="`/${ fish }.png`"
                         :alt="fish"
                         class="w-24"
                         :class="{ 'drop-shadow-[0_6px_4px_rgba(0,200,255,0.8)]': type === fish }"
                    />
                  </label>
                </li>
              </ul>
            </div>
            <div class="mb-4">
              <label for="name" class="block mb-2">Name</label>
              <input type="text" v-model="name" class="w-full border border-gray-300 rounded-lg p-2">
            </div>
            <div class="flex justify-end">
              <button type="button"
                      @click="closeModal"
                      class="bg-red-500 hover:bg-red-600 text-white rounded-lg mr-2 px-4 py-2"
              >
                Cancel
              </button>
              <button type="submit" class="bg-blue-500 hover:bg-blue-600 text-white rounded-lg px-4 py-2">Add</button>
            </div>
          </form>
        </div>
      </div>
    </template>
  </transition>
</template>
<style scoped>
.overlay-enter-active,
.overlay-leave-active {
  transition: opacity 0.5s ease;
}

.overlay-enter-from,
.overlay-leave-to {
  opacity: 0;
}

.modal-enter-active,
.modal-leave-active {
  transition: transform 0.5s ease, opacity 0.5s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
  transform: translate(-50%, calc(-50% - 30px));
}
</style>
