<script setup>
import { ref, reactive, watch, onMounted, onBeforeUnmount } from 'vue'
import Rater from './components/Rater.vue'

const email = ref('')
const round = ref(null)
let done = false
const totalRound = import.meta.env.VITE_ROUNDS_PER_RECORD
const meta = {
  apps: ['gray', 'mono', 'style'],
  types: ['cover', 'message'],
  methods: ['baseline', 'ours'],
}

function getRandomInt(max) {
  return Math.floor(Math.random() * max);
}

function start() {
  if (email.value === '') {
    return
  }
  round.value = 1
}

const trial = reactive({})
function newTrial() {
  trial.app = meta.apps[getRandomInt(meta.apps.length)]
  trial.type = meta.types[1] // no cover, only message
  trial.method = meta.methods[getRandomInt(meta.methods.length)]
  trial.sample = getRandomInt(30)
}
newTrial()

const thankYou = reactive({
  show: false,
  countdown: 10
})
function submit(score) {
  fetch(
    `${import.meta.env.VITE_GOOGLE_FORM}?` +
    `entry.1874024667=${email.value}&` +
    `entry.97356366=${JSON.stringify(trial)}&` +
    `entry.1658723679=${score}`
    , { mode: 'no-cors' })

  round.value += 1
  done = round.value > totalRound
  if (done) {
    // set a timer to reload the page
    thankYou.show = true
    setInterval(() => thankYou.countdown--, 1000)
  }
  newTrial()
}
watch(thankYou, (newThankYou, _) => {
  if (newThankYou.countdown === 0) {
    location.reload()
  }
})

const lang = ref('en-US')
onMounted(() => {
  lang.value = navigator.language || navigator.userLanguage
})

// sync width of the header and main content on mobile device
function onResize() {
  const appDOM = document.getElementById('app')
  const headerDiv = document.getElementById('header-div')
  if (headerDiv !== null) {
    if (window.matchMedia('(max-width: 1023px)').matches) {
      headerDiv.style.width = appDOM.offsetWidth + 'px'
    } else {
      headerDiv.style.width = null
    }
  }
}

const resizeObserver = new ResizeObserver(onResize)
onMounted(() => {
  resizeObserver.observe(document.getElementById('app'))
})
onBeforeUnmount(() => {
  resizeObserver.unobserve(document.getElementById('app'))
})

</script>

<template>
  <div v-if="thankYou.show">
    <template v-if="lang === 'zh-TW'">
      感謝您的填寫，我們已收到您的意見。 本頁面將在 {{ thankYou.countdown }} 秒後重新載入，
      您可以再次使用同一個 email 填寫。
    </template>
    <template v-else>
      We have received your submission. Thank you! This page will reload in
      {{ thankYou.countdown }} seconds. You can enter the same email for a new submission.
    </template>
  </div>
  <template v-else>
    <div class="main">
      <div v-if="round === null">
        <div class="lang-selector">
          <span :class="{ green: lang === 'zh-TW' }" @click="lang = 'zh-TW'">中文</span> /
          <span :class="{ green: lang !== 'zh-TW' }" @click="lang = 'en-US'">Eng</span>
        </div>
        <h1 class="green">ReVP</h1>
        <p>
          <template v-if="lang === 'zh-TW'">
            感謝您參加本次測驗，本測驗共30題，我們將抽出五位完成作答的幸運兒，贈與超商禮券100元。
            您可以提供 email 讓我們在抽到您時通知您。如中途離開，本測驗不保留您未完成的作答。
          </template>
          <template v-else>
            Thanks for your interest in participating this test. The test contains 30 questions.
            We have $100 NTD coupon giveaway to 5 random selected participates who complete the test.
            You're welcome to fill in your email address so we can reach you if you won the lottery.
            If you leave the test half-way, this site will not keep your record.
          </template>
        </p>

        <label for="email" class="green">
          <template v-if="lang === 'zh-TW'"> 請輸入您的 email 來開始作答 </template>
          <template v-else>
            Please enter your email to get started.
          </template>
        </label>

        <div>
          <input type="email" id="email" v-model="email" v-on:keyup.enter="start" />
          <button @click="start">
            <template v-if="lang === 'zh-TW'"> 送出 </template>
            <template v-else> Submit </template>
          </button>
        </div>
      </div>
      <div v-else>
        <header>
          <div id="header-div">
            <div class="lang-selector">
              <span :class="{ green: lang === 'zh-TW' }" @click="lang = 'zh-TW'">中文</span> /
              <span :class="{ green: lang !== 'zh-TW' }" @click="lang = 'en-US'">Eng</span>
            </div>
            <h1 class="green">ReVP</h1>
            <h3>
              <template v-if="lang === 'zh-TW'"> 問題 </template>
              <template v-else> Question </template>
              {{ round }}/{{ totalRound }}
            </h3>
          </div>
        </header>

        <p class="instruction">
          <template v-if="lang === 'zh-TW'">
            請將這兩張圖片由 <span class="green">1 至 5 評分 (最不相似到最相似)</span>。
          </template>
          <template v-else>
            Given two images, please rate these images <span class="green">from 1 to 5 (disimilar to similar)</span>.
          </template>
        </p>
      </div>
    </div>
    <div v-if="round !== null">
      <Rater :trial="trial" :lang="lang" @submit="submit" />
    </div>
  </template>

</template>

<style scoped>
.lang-selector {
  float: right;
  /*
  position: fixed;
  top: 10px;
  right: 20px;
  z-index: 999;
*/
}

.lang-selector span:hover {
  cursor: pointer;
}

input {
  border: 1px solid #666;
  height: 25px;
  border-radius: 5px;
  margin: 0 5px 0 0;
  font-size: 16px;
}

button {
  height: 25px;
}

p {
  margin: 10px 0 10px 0;
}

@media (min-width: 1024px) {
  .main {
    padding-right: var(--section-gap);
  }
}

@media (max-width: 1023px) {
  header {
    position: fixed;
    top: 0;
    right: 0;
    width: 100%;
    background: var(--color-background-soft);
    z-index: 1;
    display: flex;
    place-items: center;
  }

  header>div {
    padding: 0 2rem 0.5rem 2rem;
    margin: 0 auto;
  }

  p.instruction {
    margin-top: 150px;
  }
}
</style>
