<template>
  <main class="bg-dark"
        style="width:100vw;
               height:100vh;">
    <div class="position-absolute-center">

      <template v-if="isLoaded">
        <!-- 播放器 -->
        <div class="mb-15">
          <!-- 使用 Bootstrap 進度條 -->
          <div class="progress mb-10"
               @mousedown="clickChangeMusicPsotion"
               @mousemove="moveChangeMusicPsotion"
               style="height:30px;">
            <div class="progress-bar progress-bar-striped progress-bar-animated"
                 role="progressbar"
                 :aria-valuenow="progressBarValue"
                 aria-valuemin="0"
                 aria-valuemax="100"
                 :style="`width: ${progressBarValue}%`">
              {{ progressBarValue.toFixed(2) }}%
            </div>
          </div>
          <!-- 自己刻進度條 -->
          <div class="position-relative">
            <!-- 進度條 (百分比使用偽元素) -->
            <progress class="w-100 percentage"
                      id="musicProgress"
                      ref="progressRef"
                      :data-progress="`${progressBarValue.toFixed(2)}%`"
                      :value="progressBarValue" max="100"
                      @mousedown="clickChangeMusicPsotion"
                      @mousemove="moveChangeMusicPsotion"
                      style="height:50px;">
            </progress>
          </div>
          <!-- 當前播放進度時間 (例 => 00:37) -->
          <p class="d-flex align-items-center text-light">
            <span class="text-20">{{ currentTime }}</span>
          </p>
        </div>
        <!-- 控制器 -->
        <div class="d-flex algin-items-center">
          <!-- 播放、暫停 -->
          <button type="button"
                  class="btn btn-primary rounded-pill"
                  @click="togglePlayback">
            {{ isPlaying?'暫停':'播放' }}
          </button>
          <!-- 重置 -->
          <button type="button"
                  class="btn btn-secondary rounded-pill mx-5"
                  @click="resetMp3">
            重置
          </button>
          <!-- 音量 -->
          <div class="d-flex flex-column algin-items-center">
            <label for="customRange3"
                   class="d-flex justify-content-between form-label text-light text-18 text-center mb-0">
              <span>音量: </span>
              <span class="me-3">{{ volume }}</span>
            </label>
            <input type="range" class="form-range"
                   min="0" max="100" step="1"
                   v-model="volume"
                   @input="setVolume"
                   style="width:100px;">
          </div>
          <!-- 速率 -->
          <div class="d-flex flex-column algin-items-center ms-8 me-5">
            <label for="customRange3"
                   class="d-flex justify-content-between form-label text-light text-18 text-center mb-0">
              <span>速率: </span>
              <span class="me-3">{{ rate }}</span>
            </label>
            <input type="range" class="form-range"
                   min="1" max="5" step="1"
                   v-model="rate"
                   @input="setRate"
                   style="width:100px;">
          </div>
          <!-- 靜音 -->
          <button type="button"
                  class="btn btn-secondary rounded-pill mx-5"
                  @click="toggleMuteState"
                  style="width:92px;">
            {{ isMute?'取消靜音':'靜音' }}
          </button>
        </div>
        <ul class="text-light"
            style="margin-top:50px;
                   padding:20px;
                   background-color: #82777759;">
          <li>正在播放中 playing: {{ myMp3?.playing() }}</li>
          <li>音樂總長度 duration: {{ duration }} (秒)</li>
          <li>當前播放位置 seek: {{ seek>0?seek.toFixed(2):0 }}</li>
        </ul>
      </template>

      <template v-else>
        <span class="text-light">音樂載入中...</span>
      </template>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Howl, Howler } from 'howler'

console.log('Howl', Howl)
console.log('Howler', Howler)

const myMp3 = ref(null)
const isLoaded = ref(false)
const isPlaying = ref(false)
const isMute = ref(false)
const volume = ref(100)
const rate = ref(1)
const duration = ref(0)
const seek = ref(0)
const seekInterval = ref(null)
const progressBarValue = ref(0)
const progressRef = ref(null)
const currentTime = ref('00:00')

onMounted(() => {
  howlerInit()
})

function howlerInit () {
  myMp3.value = new Howl({
    // src: ['sound.webm', 'sound.mp3', 'sound.wav'],
    src: [import.meta.env.VITE_APP_MP3_URL]
    // src: ['assets/mp3.mp3'] //! GitHub Pages 開這個
    // src: ['src/assets/files/mp3.mp3'] //! 本地端開這個
    // autoplay: true,
    // loop: true,
    // volume: 0.5,
    // html5: true // 串流音訊（用於即時音訊或大檔案）
    // sprite: {
    //   blast: [0, 3000],
    //   laser: [4000, 1000],
    //   winner: [6000, 5000]
    // }
  })

  // 事件註冊
  myMp3.value.on('play', function () {
    console.log('play! 音樂開始播放')
    isPlaying.value = true
    duration.value = myMp3.value.duration()

    if (seekInterval.value) return
    seekInterval.value = setInterval(() => {
      seek.value = myMp3.value.seek()
      updateCurrentTime()
    }, 1000)
  })
  myMp3.value.on('end', function (id) {
    console.log('end 音樂播放結束')
    isPlaying.value = false
    seekInterval.value = null
  })
  myMp3.value.on('load', function () {
    console.log('load 音樂已載入')
    isLoaded.value = true
    // duration.value = myMp3.value.duration()
  })
  myMp3.value.on('loaderror', function (id, error) {
    console.error('Error loading sound:', error)
    console.log('loaderror 載入音樂時發生錯誤')
  })
  myMp3.value.on('playerror', function (id, error) {
    console.error('Error playing sound:', error)
    console.log('playerror 播放音樂時發生錯誤')
  })
  myMp3.value.on('play', function (id) {
    console.log('play 音樂開始播放，ID:', id)
  })
  myMp3.value.on('pause', function (id) {
    console.log('pause 音樂暫停，ID:', id)
  })
  myMp3.value.on('stop', function (id) {
    console.log('stop 音樂停止，ID:', id)
  })
  myMp3.value.on('volume', function (id) {
    console.log('volume 音量變更，ID:', id)
  })
  myMp3.value.on('rate', function (id) {
    console.log('rate 播放速率變更，ID:', id)
  })
  myMp3.value.on('seek', function (id) {
    console.log('seek 播放位置變更，ID:', id)
  })
  myMp3.value.on('fade', function (id) {
    console.log('fade 淡入淡出完成，ID:', id)
  })
  myMp3.value.on('mute', function (id) {
    console.log('mute 靜音事件，ID:', id)
  })
  myMp3.value.on('unlock', function () {
    console.log('unlock 音訊已解鎖')
  })
}
function togglePlayback () {
  const method = isPlaying.value ? 'pause' : 'play'
  myMp3.value[method]()
  isPlaying.value = !isPlaying.value
}
function toggleMuteState () {
  isMute.value = !isMute.value
  myMp3.value.mute(isMute.value)
}
function setVolume (e) {
  myMp3.value.volume(e.target.value / 100)
}
function setRate (e) {
  myMp3.value.rate(e.target.value)
}
function resetMp3 () {
  myMp3.value.stop()
  isPlaying.value = false
}
function updateCurrentTime (e) {
  // progress
  const progress = myMp3.value.seek() / myMp3.value.duration()
  progressBarValue.value = progress * 100

  // 當前播放的時間  例 00:37
  const curTime = parseInt(myMp3.value.seek())
  const minutes = Math.floor(curTime / 60)
  const seconds = Math.floor(curTime % 60)
  const m = `${minutes < 10 ? '0' + minutes : minutes}`
  const s = `${seconds < 10 ? '0' + seconds : seconds}`
  currentTime.value = `${m}:${s}`
}
function clickChangeMusicPsotion (e) {
  // 計算滑鼠點擊位置相對於進度條左側的偏移量
  const clickPosition = e.clientX - progressRef.value.getBoundingClientRect().left
  const progressBarWidth = progressRef.value.offsetWidth
  // 進度條的百分比 = 當前點擊的位置 除以 進度條的寬度
  const clickPercentage = clickPosition / progressBarWidth
  // 改變百分比的值  例: 15%
  progressBarValue.value = clickPercentage * 100
  // 改變播放位置
  myMp3.value.seek(myMp3.value.duration() * clickPercentage)
  updateCurrentTime()
}
function moveChangeMusicPsotion (e) {
  // 判斷是否正在拖曳（按住左鍵的狀態下移動）
  if (e.buttons === 1) {
  // 計算滑鼠點擊位置相對於進度條左側的偏移量
    const dragPosition = e.clientX - progressRef.value.getBoundingClientRect().left
    const progressBarWidth = progressRef.value.offsetWidth
    // 進度條的百分比 = 當前點擊的位置 除以 進度條的寬度
    const dragPercentage = dragPosition / progressBarWidth
    // 改變百分比的值  例: 15%
    progressBarValue.value = dragPercentage * 100

    myMp3.value.seek(myMp3.value.duration() * dragPercentage)
    updateCurrentTime()
  }
}

</script>

<style lang='scss' scope>
.percentage {
  &::before {
    content: attr(data-progress);
    display: block;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-60%);
    font-size: 20px;
    color: #000000;
    font-weight: 900;
  }
}
</style>
