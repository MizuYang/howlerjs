<template>
  <main class="bg-gainsboro"
        style="width:100vw;
               height:100vh;">
    <div class="position-absolute-center">
      <!-- 播放、暫停 -->
      <button type="button"
              class="btn btn-primary"
              @click="togglePlayback">
        {{ isPlaying?'暫停':'播放' }}
      </button>
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

</script>

<style lang='scss' scope></style>
