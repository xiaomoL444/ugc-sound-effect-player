<template>
    <div class="player">
        <button @click="togglePlay">{{ isPlaying ? '暂停' : '播放' }}</button>
        <button @click="prevTrack">上一首</button>
        <button @click="nextTrack">下一首</button>

        <div v-if="currentTrackSrc">
            <p>歌曲id: {{ currentTrackSrc.split('/').pop().split('.')[0] }}</p>
        </div>

        <!-- 时间进度 -->
        <div>
            <span>{{ formatTime(currentTime) }}</span>
            <input type="range" :max="duration" step="0.1" v-model.number="currentTime" @input="seek" />
            <span>{{ formatTime(duration) }}</span>
        </div>

        <!-- 播放速度 -->
        <div>
            <label>速度: {{ playbackRate }}x</label>
            <input type="range" min="0.5" max="2" step="0.1" v-model.number="playbackRate"
                @input="changePlaybackRate" />
        </div>

        <!-- 音量 0~200% -->
        <div>
            <label>音量: {{ Math.round(volume * 100) }}%</label>
            <input type="range" min="0" max="2" step="0.01" v-model.number="volume" @input="changeVolume" />
        </div>

        <!-- 音频元素 -->
        <audio ref="audioRef" @timeupdate="updateTime" @loadedmetadata="loadMetadata"></audio>
    </div>
</template>

<script setup>
import { ref, watch, computed, onMounted } from 'vue'

const props = defineProps({
    BGMList: { type: Array, default: () => [] },
    // currentIndex: { type: Number, default: 0 }
})
// const currentIndex = defineModel();
const audioRef = ref(null)
const isPlaying = ref(false)
const currentTrack = defineModel()
const currentTime = ref(0)
const duration = ref(0)
const playbackRate = ref(1)
const volume = ref(1)

// Web Audio API
let audioCtx = null
let sourceNode = null
let gainNode = null

const currentTrackSrc = computed(() => props.BGMList[currentTrack.value] || null)

// 初始化 Web Audio API，只创建一次 sourceNode
function initAudio() {
    const audio = audioRef.value
    if (!audio || !currentTrackSrc.value) return
    if (!audioCtx) {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    }

    if (!sourceNode) {
        sourceNode = audioCtx.createMediaElementSource(audio)
        gainNode = audioCtx.createGain()
        gainNode.gain.value = volume.value
        sourceNode.connect(gainNode).connect(audioCtx.destination)
    }
}
let firstPlay = false;
// 播放 / 暂停
function togglePlay() {
    const audio = audioRef.value
    if (!audio || !currentTrackSrc.value) return
    if (!audioCtx) initAudio()
    if (audioCtx.state === 'suspended') audioCtx.resume()
    if (isPlaying.value) audio.pause()
    else {
        audio.play()
        firstPlay = true;
    }
    isPlaying.value = !isPlaying.value
}

// 切歌，不重新创建 sourceNode
function resetAudio() {
    const audio = audioRef.value
    if (!audio || !currentTrackSrc.value) return
    audio.pause()
    audio.currentTime = 0
    isPlaying.value = false
    audio.src = currentTrackSrc.value
    audio.load()
    if (firstPlay) audio.play().then(() => isPlaying.value = true).catch(() => { })
}

// 上一首 / 下一首
function prevTrack() {
    if (props.BGMList.length === 0) return
    currentTrack.value = (currentTrack.value - 1 + props.BGMList.length) % props.BGMList.length
    resetAudio()
}

function nextTrack() {
    if (props.BGMList.length === 0) return
    currentTrack.value = (currentTrack.value + 1) % props.BGMList.length
    resetAudio()
}

// 时间控制
function seek() {
    if (!audioRef.value) return
    audioRef.value.currentTime = currentTime.value
}

function updateTime() {
    currentTime.value = audioRef.value.currentTime
}

function loadMetadata() {
    duration.value = audioRef.value.duration
}

// 播放速度
function changePlaybackRate() {
    if (!audioRef.value) return
    audioRef.value.playbackRate = playbackRate.value
}

// 音量 0~2，通过 GainNode
function changeVolume() {
    if (!gainNode) return
    const v = Number(volume.value)
    if (isNaN(v) || !isFinite(v)) return
    gainNode.gain.value = v
}

// 格式化时间 mm:ss
function formatTime(sec) {
    const m = Math.floor(sec / 60)
    const s = Math.floor(sec % 60)
    return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`
}

// 监听父组件控制索引
watch(() => currentTrack.value, (newIndex) => {
    if (newIndex >= 0 && newIndex < props.BGMList.length) {
        currentTrack.value = newIndex
        resetAudio()
    }
})

// 监听 BGMList 变化
// watch(() => props.BGMList, (newList) => {
//     if (!newList || newList.length === 0) {
//         audioRef.value?.pause()
//         isPlaying.value = false
//         currentTrack.value = 0
//         currentTime.value = 0
//     } else if (currentTrack.value >= newList.length) {
//         currentTrack.value = newList.length - 1
//         resetAudio()
//     } else {
//         resetAudio()
//     }
// }, { deep: true })

onMounted(() => {
    // 这时 audio 元素已经挂载
    audioRef.value.currentTime = 0
    initAudio()
    if (currentTrackSrc.value) resetAudio()
})
</script>

<style>
.player {
    display: flex;
    flex-direction: column;
    gap: 10px;
    width: 400px;
    margin: 20px auto;
}

input[type='range'] {
    width: 100%;
}

button {
    cursor: pointer;

    border-radius: 10px;
    border: none;
    /* 去掉所有边框 */
    border-bottom: 1px solid #887BD7;
    font-size: 1.2rem;
    padding: 0.5rem;
}
</style>