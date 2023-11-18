<script setup>
import { ref, reactive, onMounted, onUnmounted, shallowRef, computed } from 'vue';

const props = defineProps({
  id: { type: String, required: true },
  name: { type: String, required: true },
  type: { type: String, required: true }
});
const emit = defineEmits(['deleteFish']);
const directions = {
  x: ['left', 'right'],
  y: ['top', 'bottom']
};
let timer = null;
let interval = null;

const fishDomEl = ref(null);
const width = shallowRef(window.innerWidth);
const height = shallowRef(window.innerHeight);
const fish = reactive({
  ...props,
  coords: {
    x: 0,
    y: 0
  },
  directions: {
    x: directions.x[Math.floor(Math.random() * directions.x.length)],
    y: directions.y[Math.floor(Math.random() * directions.y.length)]
  },
  speed: Math.floor(Math.random() * 5 * 1000) + 1000, // 1 - 5 seconds
  animated: false,
  hungry: 0,
  alive: true,
  goneAnimate: false,
  lifetime: Math.floor(Math.random() * 300) + 100, // 20 - 40 seconds
  image: new URL(`../public/${ props.type }.png`, import.meta.url).href
});
const fishTransition = computed(() => `transform ${ fish.alive ? fish.speed / 1000 : 0 }s linear`);

const fishLifecycle = () => {
  return setInterval(() => {
    if (fish.hungry >= 100) {
      fish.image = new URL(`../public/dead.png`, import.meta.url).href;
      fish.alive = false;

      setTimeout(() => {
        fish.goneAnimate = true;
      }, 2500);

      timer = setTimeout(() => {
        emit('deleteFish', fish.id);

        if (interval) clearInterval(interval);
        if (timer)  clearTimeout(timer);
      }, 3000);
    }

    fish.hungry++;
  }, fish.lifetime);
};

const onWindowResize = () => {
  width.value = window.innerWidth;
  height.value = window.innerHeight;
};

onMounted(() => {
  window.addEventListener('resize', onWindowResize);
  setInitialCoordinates();
  interval = fishLifecycle();
});

onUnmounted(() => {
  if (interval) clearInterval(interval);
  if (timer)  clearTimeout(timer);

  window.removeEventListener('resize', onWindowResize);
});

const feed = () => {
  if (!fish.alive) {
    return false;
  }

  fish.hungry = 0;
};

const setInitialCoordinates = () => {
  fish.coords.x = width.value / 2 - fishDomEl.value.clientWidth / 2;
  fish.coords.y = height.value / 2 - fishDomEl.value.clientHeight / 2;

  setTimeout(() => {
    fish.animated = true;
    requestAnimationFrame(moveFish);
  }, 4);
};

let lastInvocationTime = 0;
const moveFish = () => {
  if (!fish.alive) {
    return setRealTransform();
  }

  const currentInvocationTime = Date.now();

  if (currentInvocationTime - lastInvocationTime >= fish.speed) {
    lastInvocationTime = currentInvocationTime;

    setX();
    setY();
  }

  return requestAnimationFrame(moveFish);
};

const setX = (step = 100) => {
  let direction = fish.directions.x;

  if ((fish.coords.x + step + fishDomEl.value.clientWidth / 2) > width.value - fishDomEl.value.clientWidth) {
    direction = 'left';
  } else if (fish.coords.x - step < fishDomEl.value.clientWidth) {
    direction = 'right';
  }

  fish.coords.x += step * (direction === 'left' ? -1 : 1);
  fish.directions.x = direction;
};

const setY = (step = 100) => {
  let direction = fish.directions.y;

  if ((fish.coords.y + step + fishDomEl.value.clientHeight) > height.value) {
    direction = 'top';
  } else if (fish.coords.y - step < 0) {
    direction = 'bottom';
  }

  fish.coords.y += step * (direction === 'top' ? -1 : 1);
  fish.directions.y = direction;
};

/**
 * This function is used to set the real coordinates of the fish.
 * It's necessary because the fish is animated with CSS transform property.
 * The transform properties are updated in moveFish() function.
 * moveFish function is called with requestAnimationFrame() function.
 * And with a delay of the fish speed.
 * Then the fish's moving is animated just by transition, not by transform.
 * And when the fish is died, it's still animated by transition.
 * We have to stop it and set the real coordinates.
 */
const setRealTransform = () => {
  const style = window.getComputedStyle(fishDomEl.value);
  const transformMatrix = style.getPropertyValue('transform');
  let values = transformMatrix.split('(')[1];

  values = values.split(')')[0];
  values = values.split(',');

  const [_0, _1, _2, _3, translateX, translateY] = values;

  fish.coords.x = parseInt(translateX);
  fish.coords.y = parseInt(translateY);
};
</script>

<template>
  <button @click="feed"
          ref="fishDomEl"
          class="absolute z-20"
          :class="{
            'pointer-events-none': !fish.alive,
            '!transition-opacity duration-500 opacity-0': fish.goneAnimate
          }"
          :style="{
            transform: `translate(${fish.coords.x}px, ${fish.coords.y}px)`,
            transition: fish.animated ? fishTransition : false
          }"
  >
    <transition name="hungry">
      <span v-if="fish.hungry > 50 && fish.alive"
            class="absolute -top-[10px] left-1/2 block w-full p-2 rounded-full bg-white transform -translate-x-1/2 -translate-y-full"
      >
        <span class="hungry-arrow"></span>
        Feed me<span class="text-red-500">!</span>
      </span>
    </transition>
    <img :src="fish.image"
         :alt="fish.name"
         :title="fish.name"
         class="block w-24 transform"
         :class="{
           '-scale-x-100': fish.directions.x === 'left'
         }"
    />
    <span class="block mt-1 p-1 rounded-t bg-[rgba(0,0,0,.7)] text-white text-center"
          :class="{
            'rounded-t': fish.alive,
            'rounded': !fish.alive
          }">
      {{ fish.name }}
    </span>
    <span v-if="fish.alive"
          class="relative overflow-hidden block rounded-b bg-[rgba(0,0,0,.7)]"
    >
      <span class="block w-full h-1 transition-transform origin-left transition-colors duration-100 ease-in-out"
            :style="{ transform: `scaleX(calc(${fish.hungry} / 100))` }"
            :class="{
              'bg-green-400': fish.hungry <= 20,
              'bg-green-300': fish.hungry > 20 &&  fish.hungry <= 30,
              'bg-yellow-200': fish.hungry > 30 &&  fish.hungry <= 40,
              'bg-yellow-300': fish.hungry > 40 &&  fish.hungry <= 50,
              'bg-yellow-400': fish.hungry > 50 &&  fish.hungry <= 60,
              'bg-orange-300': fish.hungry > 60 &&  fish.hungry <= 70,
              'bg-red-300': fish.hungry > 70 &&  fish.hungry <= 80,
              'bg-red-400': fish.hungry > 80 &&  fish.hungry <= 100
            }"
      >
        <span class="sr-only">Hungry {{ fish.hungry }}</span>
      </span>
    </span>
  </button>
</template>

<style scoped>
.hungry-enter-active,
.hungry-leave-active {
  transition: opacity 0.5s ease;
}

.hungry-enter-from,
.hungry-leave-to {
  opacity: 0;
}

.hungry-arrow::before {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  border: 6px solid transparent;
  border-color: transparent white transparent transparent;
  transform: translateX(-50%) rotate(-90deg);
}
</style>
