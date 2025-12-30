<template>
    <div ref="root" class="ElementBox">
        <div style="  
        display: flex;                  /* 开启 Flex 布局 */
  align-items: center;            /* 垂直居中 */">
            <!-- <button class="btn">复制名称</button> -->
            <div :class="['ElementBoxTitle', 'no-select']" v-on:click="Clipboard(name)">{{ name }}</div>

        </div>
        <div :class="['ElementContent', 'no-select']" v-on:click="Clipboard(id)">
            id:{{ id }} / {{ duration }}s
        </div>
    </div>
</template>

<script setup>
import { defineProps,ref } from 'vue';
import { toast } from 'vue-sonner';

const root = ref(null)
defineExpose({ root })  // 把 root 暴露给父组件

const props = defineProps({
    id: String,
    name: String, // 声明父组件传递的 prop
    duration: String
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