<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue';

const px = ref(0);
const py = ref(0);
const ps = ref(0);

onMounted(() => {
    window.addEventListener('mousemove', (ev) => {
        px.value = ev.pageX;
        py.value = ev.pageY;
    });

    window.addEventListener('mousedown', (ev) => {
        px.value = ev.pageX;
        py.value = ev.pageY;
        ps.value = 1.0;
    });

    window.addEventListener('mouseup', () => {
        ps.value = 0.0;
    });
})

onUnmounted(() => {
    window.removeEventListener('mousemove', () => {});
    window.removeEventListener('mousedown', () => {});
    window.removeEventListener('mouseup', () => {});
})

const st = computed(() => {
    return {
        left: `${px.value}px`,
        top: `${py.value}px`,
        transform: `translateX(-50%) translateY(-50%) scale(${ps.value}) rotate(${-ps.value * 5}deg)`
    };
})
</script>

<template>
    <div class="indicator" :style="st">

    </div>
</template>

<style scoped>
.indicator {
    position: absolute;
    width: 6rem;
    height: 60rem;
    background-color: #0000;
    backdrop-filter: invert();
    z-index: 10000000;
    /* border-radius: 100%; */
    pointer-events: none;
    transform: translateX(-50%) translateY(-50%) scale(1.0);
    transition: all 0.05s cubic-bezier(0.47, 0, 0.745, 0.715);
}
</style>