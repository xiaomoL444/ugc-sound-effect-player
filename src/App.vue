<template>
  <Toaster position="top-center" />
  <div style="display: flex; flex-direction: column;height:100vh;">
    <div style="/* text-align: center; */
  display: flex;
  /* justify-content: center; */
  /* 水平居中 */
  align-items: center;
    flex: 1.2;
  padding-left: 2rem;   background: linear-gradient(90deg, #6a5acd, #00bfff);">
      <img :src="imgSrc" style="height: 2.5rem; width: 2.5rem; margin-right: 1rem;">
      <h2 class="title">音效播放器</h2>
    </div>
    <div class="container">
      <div class="block" style="display: flex;flex-direction: column; flex:1;  width: 20rem;      /* 固定宽度 */
  flex-shrink: 0;    /* 不允许被压缩 */">
        <Title :title="'选择音乐'"></Title>

        <div style=" margin: 0.25rem; display: flex;
  flex-direction: column;
  gap: 12px; /* 子元素上下间距 */ overflow-y: auto;">
          <div style="display: flex; flex-direction: row;">搜索（可搜索id）：<input v-model="search"></div>
          <RecycleScroller :items="rows" :item-size="120" key-field="" height="400" v-slot="{ item: row, index }">
            <div class="selectBox" :ref="el => itemRefs[index] = el">
              <SelectBox v-for="key in row" :key="key" :id="key" :name="dataJson[key].name"
                :duration="dataJson[key].duration" :class="{ 'active': currentID === key }" @click="SelectSound(key)" />
            </div>
          </RecycleScroller>

          <!-- <SelectBox :id="key" :name="dataJson[key].name" :duration="dataJson[key].duration"
            :class="{ active: currentID === key }" @click="SelectSound(key)" :ref="el => itemRefs[index] = el" /> -->

          <!-- <div class="selectBox">
            <SelectBox v-for="(key, index) in Object.keys(dataJson)" :key="key" @click="SelectSound(key)"
              :ref="el => itemRefs[index] = el" :id="key" :name="dataJson[key]['name']"
              :class="{ 'active': currentID === key }" :duration="dataJson[key]['duration']" />
          </div> -->

        </div>
      </div>
      <div class="block" style="  overflow-y: scroll;flex: 1;">
        <Title :title="'播放器'"></Title>
        <MusicBox :BGMList="musicList" v-model="current_index" />
      </div>
      <!-- <pre class="output">
{{ pixels }}
    </pre> -->
    </div>
  </div>
</template>

<script setup>

/* eslint-disable */

function getKey(item) {
  return item
}

import logo from '@/assets/7.png'  // @ = src/
const imgSrc = logo

import { computed, ref, watch } from 'vue'
import SelectBox from './components/SelectBox.vue'
import { Toaster } from 'vue-sonner'
import 'vue-sonner/style.css'
import Title from './components/TitleBox.vue'

const canvas = ref(null)
const imgUrl = ref(null)
const pixelHeight = ref(20)//像素画高
const pixelWidth = ref(20) //像素画宽
const maxPixelWidth = ref(20) //每行最大像素画宽
const pixels = ref([])

const itemRefs = ref([])   // 存每个 div 的 DOM 元素

import data from '@/assets/SoundEffectData.json'
import { toast } from 'vue-sonner'
import MusicBox from './components/MusicBox.vue'
import { RecycleScroller } from 'vue-virtual-scroller'

const search = ref('');

const keys = computed(() => Object.keys(dataJson).filter(key => {
  const value = dataJson[key]
  return value.name.includes(search.value) || value.id.includes(search.value)
}))

const rows = computed(() => {
  const result = []
  for (let i = 0; i < keys.value.length; i += 3) {
    result.push(keys.value.slice(i, i + 3))
  }
  console.log(result)
  return result
})

const dataJson = data

const musicList = computed(() => keys.value.map(x => `${process.env.BASE_URL}sound/${x}.mp3`))

const currentID = ref(0)
const current_index = ref(0);
function SelectSound(id) {
  console.log(`选择了${id}`)
  currentID.value = id
  current_index.value = keys.value.indexOf(id);
}

function rgbaToHex(r, g, b, a = 255) {
  const toHex = v => v.toString(16).padStart(2, '0').toUpperCase()
  return `#${toHex(r)}${toHex(g)}${toHex(b)}${toHex(a)}`
}
// 监听 currentId 变化，滚动到可视
watch(currentID, (newId) => {
  const index = Math.floor(keys.value.indexOf(newId) / 3)
  console.log(index)
  if (index !== -1 && itemRefs.value[index]) {
    itemRefs.value[index].scrollIntoView({ behavior: 'smooth', block: 'nearest' })
  }
})

watch(current_index, (new_index) => {
  currentID.value = keys.value[new_index]
})
</script>

<style scoped>
@import "vue-virtual-scroller/dist/vue-virtual-scroller.css";

.selectBtn {
  color: white;
  background-color: #4FACFE;
  border: #000;
  border-radius: 10px;
  font-size: 1.2rem;
  padding: 1rem;
  cursor: pointer;
  transition:
    background-color 0.3s ease,
    transform 0.2s ease-out,
    color 0.3s ease;
}

.selectBtn:hover {
  background-color: #337ecc;
  /* 更深一点的颜色 */

}

.selectBox {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* 每行 3 个 */
  gap: 10px;
}

.title {
  font-weight: 700;
  /* 粗体 */
  font-size: 2rem;
  /* 放大一点好看 */
  /* background: linear-gradient(0deg, #6a5acd, #00bfff); */
  /* 紫 → 蓝 */
  background-color: white;
  -webkit-background-clip: text;
  /* 背景裁剪到文字 */
  color: transparent;
  /* 隐藏文字颜色，让渐变显现 */
  -webkit-text-fill-color: transparent;
  /* Safari 兼容 */
}

.container {
  display: flex;
  flex-direction: row;
  /* gap: 12px; */
  flex: 18;
  height: 90vh;
}

.container .block {
  border: 1px solid #ccc;
  flex: 1;

}

.container .block .titleBlock {
  border: 1px solid #ccc;
}

.container .block .containerBlock {
  border: 1px solid #ccc;
}

.preview {
  max-width: 200px;
  border: 1px solid #ccc;
}

canvas {
  width: 200px;
  height: 200px;
  border: 1px solid #000;
  image-rendering: pixelated;
}

.output {
  max-height: 200px;
  overflow: auto;
  background: #111;
  color: #0f0;
  padding: 8px;
  font-size: 12px;
}

.outputContainer {
  margin: 0.5rem .5rem;
}

div {
  transition: background-color 0.3s ease;
  /* 过渡效果 */
}

.active {
  background-color: #d0e6ff;
}
</style>
