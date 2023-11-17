<script setup>
const props = defineProps({
  id: { type: String, required: true },
  name: { type: String, required: true },
  type: { type: String, required: true }
});

const feed = () => {
  console.log(`Feed ${props.name}!`);
};

const getFishImage = type => new URL(`../public/${ type }.png`, import.meta.url).href;
</script>

<template>
  <button @click="feed" class="relative z-10">
    <img :src="getFishImage(type)"
         :alt="name"
         :title="name"
         class="block w-24"
    />
    <span class="block mt-1 p-1 rounded bg-[rgba(0,0,0,.7)] text-white text-center">{{ name }}</span>
  </button>
</template>

<style scoped></style>
