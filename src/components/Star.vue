<template>
    <button class="star-btn">
        <svg viewBox="0 0 22 22" class="star" :class="{ active: props.active }">
            <path d="M12 17.3l-6.18 3.64 1.64-7.03L2 8.76l7.19-.61L12 2l2.81 6.15 7.19.61-5.46 5.15 1.64 7.03z"
                fill="currentColor" />
        </svg>
    </button>
</template>

<script setup>
const props = defineProps({
    active: Boolean
})

import { defineProps } from 'vue';
</script>


<style scoped>
.star-btn {
    background: none;
    border: none;
    cursor: pointer;
}

.star {
    width: 24px;
    color: #bbb;
    transition: color 0.2s, transform 0.2s;
}

.star.active {
    color: #f4c63b;
    transform: scale(1.15);
}
</style>