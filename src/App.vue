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

function getProbIndex(prob) {
  // prob for 'message'
  var sampleNum = 30
  var idxArr = new Array(sampleNum);
  idxArr.fill(1, 0, sampleNum * prob);
  idxArr.fill(0, sampleNum * prob, sampleNum);
  var idx = Math.floor(Math.random() * idxArr.length);
  return idxArr[idx];
}

const checked = ref(false)
const showWarning = ref(false)
function start() {
  if (checked.value === false) {
    showWarning.value = true
    return
  }
  showWarning.value = false
  round.value = 1
}

const trial = reactive({})
function newTrial() {
  trial.app = meta.apps[getRandomInt(meta.apps.length)]
  trial.type = meta.types[getProbIndex(0.9)] // no cover, only message
  trial.method = meta.methods[getRandomInt(meta.methods.length)]
  trial.sample = getRandomInt(30)
}
newTrial()

const thankYou = reactive({
  show: false,
  countdown: 10
})

function submit(score) {
  const keys = JSON.parse(import.meta.env.VITE_GOOGLE_FORM_ENTRIES)
  const values = [
    email.value,
    round.value,
    trial.app,
    trial.type,
    trial.method,
    trial.sample,
    score,
  ]
  let formResponse = `${import.meta.env.VITE_GOOGLE_FORM_URL}?`
  for (let i = 0; i < keys.length; i++) {
    formResponse += `${keys[i]}=${values[i]}&`
  }
  formResponse = formResponse.slice(0, -1)
  fetch(formResponse, { mode: 'no-cors' })

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
            <!-- 感謝您參加本次測驗，本測驗共30題，我們將抽出五位完成作答的幸運兒，贈與超商禮券100元。
            您可以提供 email 讓我們在抽到您時通知您。如中途離開，本測驗不保留您未完成的作答。 -->

            大家好，我們是中央研究院資訊創新研究中心陳駿丞副研究員的研究團隊。
            <br><br>
            我們正在進行 AI 生成影像與影片之使用者偏好評量相關研究。本研究旨在評估現有影像與影片生成模型的生成品質，通過主觀和客觀的評估方法，分析模型生成影像的品質，進一步了解並分析生成模型的特性。
            <br><br>
            本研究將透過此網站進行問卷調查，提供您由模型預先生成的影像或影片，並請您針對其品質及是否符合給定的生成條件進行評分。
            <br><br>
            有效填寫問卷者可選擇留下email參加抽獎，我們將抽出有效問卷的 1/20 贏取7-11百元禮券。
          </template>
          <template v-else>
            <!-- Thanks for your interest in participating this test. The test contains 30 questions.
            We have $100 NTD coupon giveaway to 5 random selected participates who complete the test.
            You're welcome to fill in your email address so we can reach you if you won the lottery.
            If you leave the test half-way, this site will not keep your record. -->

            Hello everyone, we are the research team of Associate Research Fellow Jun-Cheng Chen from the Research Center for Information Technology Innovation at Academia Sinica.
            <br><br>
            We are currently conducting research on user preference evaluation of AI-generated images and videos. The aim of this study is to assess the quality of existing image and video generation models. 
            <br><br>
            Through subjective and objective evaluation methods, we will analyze the quality of the images produced by these models to further understand and analyze their characteristics.
            <br><br>
            Participants who complete the survey accurately can choose to provide their email address for a chance to enter a raffle. We will randomly select 1 out of every 20 valid survey submissions to win a 100 NT$ 7-11 gift voucher.
          </template>
        </p>

        <div>
          <br>
          <template v-if="lang === 'zh-TW'">
            <a href="https://drive.google.com/file/d/15i3S7kWxVL5SZGJCJZ4sVtcDdbqXLppG/view" target="_blank">中央研究院人文社會科學研究對象說明同意書</a>
          </template>
          <template v-else>
            <a href="https://drive.google.com/file/d/15i3S7kWxVL5SZGJCJZ4sVtcDdbqXLppG/view" target="_blank">Academia Sinica Humanities and Social Sciences Research Subject Description and Consent Form</a>
          </template>
            <br>
          <input type="checkbox" id="checkbox" v-model="checked" />
          <template v-if="lang === 'zh-TW'"> 我已閱讀並同意 </template>
          <template v-else> I have read and agree </template>
        </div>

        <label for="email" class="green">
          <br>
          <template v-if="lang === 'zh-TW'">
            <!-- 請輸入您的 email 來開始作答 -->
            抽獎用信箱（選填，僅為抽獎通知用）
          </template>
          <template v-else>
            <!-- Please enter your email to get started. -->
            E-mail (optional, only for the lottery notification)
          </template>
        </label>

        <div>
          <input type="email" id="email" v-model="email" v-on:keyup.enter="start" />
          <button @click="start">
            <template v-if="lang === 'zh-TW'"> 下一步 </template>
            <template v-else> Next </template>
          </button>
        </div>
        <p class="green" for="" v-if="showWarning">
          <template v-if="lang === 'en-US'"> Please confirm that you've read the consent form and agree </template>
          <template v-else> 請確認已閱讀以上同意書並同意 </template>
        </p>
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
    <div v-if="round === null">
      <br>
      <div class="scrollable">
        <template v-if="lang === 'zh-TW'">
          <h3>研究參與資訊</h3>
          <ul>
            <li>資料收集：我們計劃收集100-200名受試者的生成影像或影片評分資料。若有人中途退出或數據無法使用，將遞補至收集完所需受試者為止。</li>
            <li>遞補機制：若有人中途退出或數據無法使用，將遞補至收集完所需受試者為止。</li>
            <li>目標對象：已滿 18 歲且未受監護宣告之成年人。</li>
            <li>經費來源：中研院資創中心智慧優網。</li>
          </ul>
          <br>
          <h3>資料管理與隱私保障</h3>
          <ul>
            <li>資料用途：本問卷內容僅供學術研究之用。</li>
            <li>聯絡資訊：您的 email 等聯絡資訊僅供 7-11 禮券抽獎使用，不會留下任何身份等敏感資訊。</li>
            <li>資料刪除：所有聯絡資料將於問卷調查與抽獎完畢後立即刪除。</li>
            <li>退出與刪除權利：您可在填寫問卷時隨時退出，填寫完成後也可來信 (<a href="nickchiu@citi.sinica.edu.tw">nickchiu@citi.sinica.edu.tw</a> 或 <a href="pullpull@citi.sinica.edu.tw">pullpull@citi.sinica.edu.tw</a>) 要求刪除相關資料。</li>
          </ul>
          <br>
          <h3>資料管理細節</h3>
          <ul>
            <li>不收集敏感資料：本計畫實驗不收集任何關於您的個人敏感資料，僅收集生成影像或影片的評分。</li>
            <li>資料儲存與加密：實驗收集的資料將儲存於實驗地點（中央研究院資訊創新研究中心）的資料處理電腦，並會進一步加密處理。除了評分結果與抽獎用 email，研究人員分析資料時不會取得您的身分資料。</li>
            <li>資料使用限制：所收集的資料僅供本實驗室人員學術研究使用。</li>
            <li>抽獎後刪除：email 聯絡資訊將於抽獎完成後（問卷結束後一至兩週內進行）立即刪除。</li>
            <li>資料庫管理：此計畫所收集的資料庫（生成影像或影片評分結果）為不記名資料庫，放置在主持人實驗室伺服器，資料保存和使用時間至計畫結束（2025/12/31）。若被挪作其他用途，本計畫團隊將追溯非法使用來源，以確保您的權益。計畫結束後，實驗室將銷毀相關問卷評量結果。</li>
          </ul>
          <br>
          <h3>參與權益聲明</h3>
          <ul>
            <li>研究計畫名稱：AI 生成影像與影片之使用者偏好評量</li>
            <li>編號：ASIRB-HS-24031</li>
            <li>自願參與：參加此項計畫是完全自願性質，無論參加與否均不影響您的權益。</li>
            <li>權益諮詢：若對參與研究的相關權益有疑問，可來電查詢中央研究院人文社會科學研究倫理委員會 IRB on Humanities & Social Science Research / IRB-HS，查詢電話：02-27898722。</li>
          </ul>
          <br>
          <h3>實驗進行方式</h3>
          您將對預先使用不同生成模型的生成影像或影片進行個人主觀的品質評分與排序，並評估其是否與給定的條件相符（如文字描述等條件）。您將進行 30 組的實驗（圖片兩兩為一組），大約花費 5 至 10 分鐘，但您可隨時依自己的狀態停止實驗。
        </template>
        <template v-else>
          
          <h3>Research Participation Information</h3>
          <ul>
            <li>Data Collection: We will collect image or video evaluation data from 100-200 participants. If any participant withdraws or their data becomes unusable, we will recruit additional participants until the required number is reached.</li>
            <li>Target Subject: Adults aged 18 and above who are not under guardianship.</li>
            <li>Funding Source: Intelligent Networks Program at the Research Center for Information Technology Innovation, Academia Sinica</li>
          </ul>
          <br>
          <h3>Data Management and Privacy Protection</h3>
          <ul>
            <li>Data Usage: The content of this questionnaire is solely for academic research purposes.</li>
            <li>Contact Information: Your email and other contact information will only be used for a 7-11 gift voucher lottery and will not be used to record any personal identification or sensitive information.</li>
            <li>Data Deletion: All contact information will be deleted immediately after the questionnaire survey and lottery are completed.</li>
            <li>Withdrawal and Deletion Rights: You can withdraw from the questionnaire at any time during the process. After completion, you can also request the deletion of relevant data by sending an email to <a href="nickchiu@citi.sinica.edu.tw">nickchiu@citi.sinica.edu.tw</a> or <a href="pullpull@citi.sinica.edu.tw">pullpull@citi.sinica.edu.tw</a>.</li>
          </ul>
          <br>
          <h3>Details of Data Management</h3>
          <ul>
            <li>No Collection of Sensitive Data: This project does not collect any of your personal sensitive data, only evaluations of the generated images or videos.</li>
            <li>Data Storage and Encryption: Collected data will be stored on the data processing computers at the research site (Academia Sinica Research Center for Information Technology Innovation) and will be further encrypted. Researchers analyzing the data will not have access to your identity information, except for the evaluation results and the email used for the lottery.</li>
            <li>Data Usage Restrictions: The collected data will only be used for academic research by the personnel of this laboratory.</li>
            <li>Deletion After Lottery: Email contact information will be deleted immediately after the lottery is completed (within one to two weeks after the questionnaire ends).</li>
            <li>Database Management: The database collected in this project (evaluation results of generated images or videos) is anonymous and will be stored on the principal investigator's laboratory server. Data storage and usage will continue until the end of the project (2025/12/31). If the data is used for any other purpose, our team will track the source of unauthorized use to protect your rights. After the project concludes, the laboratory will destroy the relevant questionnaire evaluation results.</li>
          </ul>
          <br>
          <h3>Statement of Participation Rights</h3>
          <ul>
            <li>Voluntary Participation: Participation in this project is entirely voluntary, and your decision to participate or not will not affect your rights in any way.</li>
            <li>ID: ASIRB-HS-24031</li>
            <li>Voluntary Participation: Participation in this project is completely voluntary, and your rights will not be affected whether you choose to participate or not.</li>
            <li>Inquiry of Rights: If you have any questions about your rights related to research participation, please call the Academia Sinica Institutional Review Board on Humanities & Social Science Research (IRB-HS) at 02-27898722.</li>
          </ul>
          <br>
          <h3>Experimental Procedure</h3>
          You will be asked to provide your subjective evaluation and ranking of the quality of pre-generated images or videos using different generation models, and to assess whether they meet the given criteria (e.g., text descriptions). You will participate in 30 sets of experiments (the images are grouped in pairs), which will take approximately 5 to 10 minutes. You can stop the experiment at any time based on your own condition.
        </template>
      </div>
    </div>
    <div v-else>
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
  .scrollable {
  width: 500px;
  height: 430px;
  overflow-y: auto; /* Vertical scrollbar */
  overflow-x: hidden; /* Hide horizontal scrollbar if not needed */
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
