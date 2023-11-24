<template>
  <main class="bg-dark"
        style="width:100vw;
               height:100vh;">
    <div class="position-absolute-center">
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
        <!-- 靜音 -->
        <button type="button"
                class="btn btn-secondary rounded-pill mx-5"
                @click="toggleMuteState"
                style="width:92px;">
          {{ isMute?'取消靜音':'靜音' }}
        </button>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Howl, Howler } from 'howler'

console.log('Howl', Howl)
console.log('Howler', Howler)

const myMp3 = ref(null)
const isPlaying = ref(false)
const isMute = ref(false)
const volume = ref(100)

onMounted(() => {
  howlerInit()
})

function howlerInit () {
  myMp3.value = new Howl({
    // src: ['sound.webm', 'sound.mp3', 'sound.wav'],
    src: ['src/assets/files/mp3.mp3'],
    // autoplay: true,
    // loop: true,
    // volume: 0.5,
    // html5: true // 串流音訊（用於即時音訊或大檔案）
    // sprite: {
    //   blast: [0, 3000],
    //   laser: [4000, 1000],
    //   winner: [6000, 5000]
    // }
    onend: function () {
      console.log('Finished!')
    }
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
function resetMp3 () {
  myMp3.value.stop()
  isPlaying.value = false
}

</script>

<style lang='scss' scope>
</style>
