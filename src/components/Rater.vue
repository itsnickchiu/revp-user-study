<script setup>
import { ref, reactive } from 'vue'
const props = defineProps({
  trial: {
    required: true
  },
  lang: {
    required: true
  }
})

const dataURL = import.meta.env.VITE_DATA_URL

const score = ref(null)

const emit = defineEmits(['submit'])
const showWarning = ref(false)
function submit() {
  // validation
  if (score.value === null) {
    showWarning.value = true
    return
  }
  showWarning.value = false
  emit('submit', score.value)
  score.value = null
}
</script>

<template>
  <h2 class="green">
    <template v-if="lang === 'en-US'"> How similar are these images? </template>
    <template v-else> 兩張圖有多相似？ </template>
  </h2>
  <div class="image-container">
    <img :src="`${dataURL}/` +
      `${trial.app}/target/` +
      `${trial.type}/` +
      `${trial.sample.toString().padStart(2, '0')}.jpg`">
    <img :src="`${dataURL}/` +
      `${trial.app}/reconst/` +
      `${trial.type}/` +
      `${trial.method}/` +
      `${trial.sample.toString().padStart(2, '0')}.jpg`">
  </div>

  <div class="score-container">

    <template v-if="lang === 'en-US'"> Dissimilar </template>
    <template v-else> 不相似 </template>
    <div class="radio-container">
      <div class="radio-item" v-for="id in 5">
        <input type="radio" :id="`radio-${id}`" :value="id" v-model="score" />
        <label :for="`radio-${id}`">{{ id }}</label>
      </div>
    </div>
    <template v-if="lang === 'en-US'"> Similar </template>
    <template v-else> 相似 </template>
  </div>

  <p class="green" v-if="showWarning">
    <template v-if="lang === 'en-US'"> Please rate before submitting </template>
    <template v-else> 請評分後再送出 </template>
  </p>
  <button @click="submit">
    <template v-if="lang === 'en-US'"> Submit </template>
    <template v-else> 送出 </template>
  </button>
</template>

<style scoped>
p {
  margin: 10px 0 10px 0;
}

.image-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
}

img {
  border: 2px solid #888;
  width: 48%;
  margin: 3px;
}

.score-container {
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
  margin-bottom: 10px;
}

.radio-container {
  display: flex;
  justify-content: space-between;
  width: 60%;
}

.radio-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

label {
  font-size: 13pt;
  cursor: pointer;
}

input {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;

  border-radius: 50%;
  width: 20px;
  height: 20px;

  border: 2px solid var(--color-text);
  transition: 0.2s all linear;
  margin-bottom: 5px;

  cursor: pointer;
}

input:checked {
  border: 6px solid var(--color-text);
  outline: unset !important
    /* I added this one for Edge (chromium) support */
}
</style>
