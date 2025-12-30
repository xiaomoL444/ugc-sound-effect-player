<template>
    <div ref="root" class="ElementBox">
        <div style="  
        display: flex;                  /* 开启 Flex 布局 */
  align-items: center;            /* 垂直居中 */">
            <!-- <button class="btn">复制名称</button> -->
            <div :class="['ElementBoxTitle', 'no-select']" v-on:click="Clipboard(name)">{{ name }}</div>

        </div>
        <div style="display: flex;flex-direction: row;   display: flex;
  justify-content: space-between; /* 两端对齐 */
  align-items: center; /* 垂直居中，可选 */">
            <div :class="['ElementContent', 'no-select']" v-on:click="Clipboard(id)">
                id:{{ id }} / {{ duration }}s
            </div>
            <div @click="like">
                <Star :active="isLike"></Star>
            </div>
        </div>
    </div>
</template>

<script setup>
import { defineProps, onMounted, ref } from 'vue';
import { toast } from 'vue-sonner';
import Star from './Star.vue';

const root = ref(null)
defineExpose({ root })  // 把 root 暴露给父组件

const props = defineProps({
    id: String,
    name: String, // 声明父组件传递的 prop
    duration: String,
    like: Boolean
})

function Clipboard(str) {
    console.log(props.content);

    navigator.clipboard.writeText(props.content)
        .then(() => {
            toast.success(` ${str} 复制成功`)
        })
        .catch(err => {
            toast.error('复制失败' + err)
        })
}

const isLike = ref(false)

onMounted(() => {
    const raw = localStorage.getItem('SoundEffectLikeSetting')
    const likeSetting = raw ? JSON.parse(raw) : {}

    isLike.value = (props.id in likeSetting) ? likeSetting[props.id] : false
})

function like() {
    const raw = localStorage.getItem('SoundEffectLikeSetting')
    const likeSetting = raw ? JSON.parse(raw) : {}
    const id = props.id;
    if (id in likeSetting) {
        likeSetting[id] = !likeSetting[id]
    }
    else {
        likeSetting[id] = true;
    }

    isLike.value = likeSetting[id]

    localStorage.setItem('SoundEffectLikeSetting', JSON.stringify(likeSetting))
}
</script>

<style scoped>
.btn {
    margin-right: 2rem;
    color: white;
    background-color: #4FACFE;
    padding: 8px 17px;

    font-size: large;

    border: 0;
    border-radius: 10px;
    transition:
        background-color 0.3s ease,
        transform 0.2s ease-out,
        color 0.3s ease;
}

.btn:hover {
    background-color: #337ecc;
    /* 更深一点的颜色 */

}

.no-select {
    user-select: none;
    /* 禁止选取文本 */
    cursor: default;
    /* 鼠标悬停时显示普通箭头，而不是文本光标 */
}
</style>