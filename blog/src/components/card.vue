<script setup>
import { ref, computed, onMounted, watch, useTemplateRef, onUnmounted } from 'vue';
import MarkdownIt from 'markdown-it'

const blogBase = '/blogs/';
const blogData = ref(null);
const blogText = computed(() => blogData.value?.text);
const blogDesc = computed(() => blogData.value?.desc.replace('\n', '<br>'));
const blogTitle = computed(() => blogData.value?.title);
const blogDate = computed(() => {
    const date = new Date(blogData.value?.timestamp);
    let result = [null, null, null];
    result[0] = date.getFullYear().toString();
    result[1] = ((month) => {
        return ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sept', 'Oct', 'Nov', 'Dec'][month];
    })(date.getMonth());
    result[2] = ((day) => {
        if ([1, 21, 31].includes(day)) return `${day}st`;
        if ([2, 22].includes(day)) return `${day}nd`;
        if ([3, 23].includes(day)) return `${day}rd`;
        return `${day}th`;
    })(date.getDate());
    return result;
});

async function loadFromSrc(src) {
    const url = `${blogBase}${src}/blog.json`
    const response = await fetch(url).then((result) => {
        if (result.ok) {
            return result.json();
        } else {
            return null;
        }
    }).catch(err => console.error(err));

    const srcUrl = response?.src;
    const text = await fetch(`${blogBase}${src}/${srcUrl}`).then((result) => {
        if (result.ok) {
            return result.text();
        } else {
            return null;
        }
    }).catch(err => console.error(err));
    response.text = new MarkdownIt().render(text);
    // console.log(text);
    return response;
}

const props = defineProps({
    src: String
});

onMounted(async () => blogData.value = await loadFromSrc(props.src));
watch(props, async (ns) => {
    console.log(ns.src);
    blogData.value = await loadFromSrc(ns.src);
    initPose();
});


const mouseX = ref(0.5);
const mouseY = ref(0.5);
const deg = ref(0.0);

const sca = ref(1.0);
const degx = ref(0.0);
const degy = ref(0.0);
const degz = ref(0.0);
const initPose = () => {
    degx.value = (Math.random() - 0.5) * 24;
    degy.value = (Math.random() - 0.5) * 24;
    degz.value = (Math.random() - 0.5) * 9;
}

const cardref = useTemplateRef('card');

let isIdle = ref(true);
let isClicked = ref(false);
let isActive = ref(false);

onMounted(initPose)

function mouseIn(ev) {
    if (isActive.value && !isIdle.value)
        return;
    const bounds = ev.target.getBoundingClientRect();
    const nx = (ev.clientX - bounds.x) / bounds.width;
    const ny = (ev.clientY - bounds.y) / bounds.height;
    mouseX.value = nx;
    mouseY.value = ny;
    sca.value = 1.05;
    isIdle.value = false;
    // cardref.value.scrollIntoView({
    //     behavior: 'smooth',
    //     block: 'center'
    // });
}

function mouseOut() {
    if (isActive.value && !isIdle.value)
        return;
    mouseX.value = 0.5;
    mouseY.value = 0.5;
    deg.value = 0.0;
    sca.value = 1.0;
    isIdle.value = true;
    // isActive.value = false;
}

function mouseDown() {
    if (isActive.value && !isIdle.value)
        return;
    sca.value = 0.9;
    isClicked.value = true;
    cardref.value.scrollIntoView({
        behavior: 'smooth',
        block: 'center'
    });
}

function mouseUp() {
    if (isActive.value && !isIdle.value)
        return;
    sca.value = 1.05;
    isClicked.value = false;
    isActive.value = true;
}

const hoverStyle = computed(() => {
    const nx = mouseX.value - 0.5;
    const ny = mouseY.value - 0.5;
    deg.value = (isClicked.value ? -25 : 25) * Math.sqrt(nx * nx + ny * ny);
    let style = {
        transform: `rotateX(${degx.value}deg) rotateY(${degy.value}deg) rotateZ(${degz.value + (isActive.value ? -5 : 0)}deg) rotate3D(${ny}, ${-nx}, 0.0, ${deg.value}deg) scale(${sca.value})`,
    };
    if (isActive.value && !isIdle.value) {
        style = { transform: `scale(${1.3}` }
    }
    return style;
});

onMounted(() => {
    window.addEventListener('mousedown', (ev) => {
        const bounds = cardref.value?.getBoundingClientRect();
        if (!bounds || ev.clientX < bounds.left || ev.clientX > bounds.right || ev.clientY < bounds.top || ev.clientY > bounds.bottom) {
            isActive.value = false;
            isIdle.value = true;
        }
    })
})

onUnmounted(() => {
    window.removeEventListener('mousedown', () => { })
})
</script>

<template>
    <div class="cardContainer" @mousemove="mouseIn" @mouseleave="mouseOut" @mousedown="mouseDown" @mouseup="mouseUp"
        :class="{ active: isActive }">
        <div ref="card" class="card" :class="{ idle: isIdle, active: isActive && !isIdle }" :style="hoverStyle">
            <p class="description" :class="{ idle: isIdle }" style="font-size: 0.5rem; line-height: 0px;">
                Burn After Read
            </p>
            <svg class="postalIcon" width="111" height="16" viewBox="0 0 111 16" fill="none"
                xmlns="http://www.w3.org/2000/svg" :class="{ idle: isIdle }">
                <path
                    d="M0.5 0.5H15.5V15.5H0.5V0.5ZM19.5 0.5H34.5V15.5H19.5V0.5ZM76.5 0.5H91.5V15.5H76.5V0.5ZM95.5 0.5H110.5V15.5H95.5V0.5ZM38.5 0.5H53.5V15.5H38.5V0.5ZM57.5 0.5H72.5V15.5H57.5V0.5Z"
                    stroke="currentColor" />
            </svg>
            <div class="indicator" :class="{ idle: isIdle, active: isActive && !isIdle }">
                <svg xmlns="http://www.w3.org/2000/svg" width="1600%" height="1600%" fill="currentColor"
                    class="bi bi-asterisk" viewBox="0 0 128 128">
                    <path
                        d="M4.5 3a2.5 2.5 0 0 1 5 0v9a1.5 1.5 0 0 1-3 0V5a.5.5 0 0 1 1 0v7a.5.5 0 0 0 1 0V3a1.5 1.5 0 1 0-3 0v9a2.5 2.5 0 0 0 5 0V5a.5.5 0 0 1 1 0v7a3.5 3.5 0 1 1-7 0z" />
                </svg>
            </div>
            <div class="headerInfo">
                <h1 class="title" :class="{ idle: isIdle }">{{ blogTitle }}</h1>
                <h2 class="date" :class="{ idle: isIdle }">
                    {{ blogDate == null ? "year" : blogDate[0]
                    }}<br>
                    {{ blogDate == null ? "mm" : blogDate[1]
                    }}<br>
                    {{ blogDate == null ? "dd" : blogDate[2] }}
                </h2>
            </div>
            <div class="content contentPlaceholder"
                style="position: relative; column-count: 2; padding: 1.2rem; margin-top: -17rem; opacity: 0; pointer-events: none;"
                v-html="blogText">
            </div>
            <p class="description" :class="{ idle: isIdle }" v-html="blogDesc">
            </p>
            <div class="contentWrapper" :class="{ active: isActive && !isIdle }">
                <div>
                    <h1 style="text-transform: uppercase;">
                        {{ blogTitle }}
                    </h1>
                    <svg width="135" height="100" viewBox="0 0 135 100" fill="none" xmlns="http://www.w3.org/2000/svg"
                        style="position: absolute; left: 96%; top: 96%;transform: translateX(-100%) translateY(-100%) rotate(5deg); opacity: 0.1;">
                        <path
                            d="M0.646999 34.0882C4.90056 31.0009 9.3378 27.8105 17.9966 27.7628C26.651 27.7143 31.025 30.857 35.2352 33.8949C39.4038 36.9051 43.3429 39.7454 51.4349 39.6952C59.5235 39.6536 63.5059 36.7699 67.7135 33.7147C71.9679 30.63 76.3723 27.437 85.0319 27.3894C93.688 27.3417 96.8455 30.4931 101.053 33.5301C105.219 36.5403 108.754 39.3798 116.053 39.3416M0.646999 48.7317C4.90056 45.6444 9.3378 42.4532 17.9966 42.4064C26.651 42.357 31.025 45.4997 35.2352 48.5385C39.4038 51.5477 43.3429 54.3881 51.4349 54.3387C59.5235 54.2971 63.5059 51.4134 67.7135 48.3574C71.9679 45.2718 76.3723 42.0797 85.0319 42.0329C93.688 41.9853 96.8455 45.1358 101.053 48.1728C105.219 51.183 108.754 54.0233 116.053 53.9852M0.646999 62.2488C4.90056 59.1615 9.3378 55.9711 17.9966 55.9235C26.651 55.875 31.025 59.0177 35.2352 62.0556C39.4038 65.0657 43.3429 67.906 51.4349 67.8558C59.5235 67.8142 63.5059 64.9306 67.7135 61.8753C71.9679 58.7907 76.3723 55.5977 85.0319 55.55C93.688 55.5024 96.8455 58.6538 101.053 61.6908C105.219 64.7009 108.754 67.5404 116.053 67.5023M0.646999 76.8923C4.90056 73.8051 9.3378 70.6138 17.9966 70.567C26.651 70.5176 31.025 73.6604 35.2352 76.6991C39.4038 79.7084 43.3429 82.5487 51.4349 82.4993C59.5235 82.4577 63.5059 79.5741 67.7135 76.518C71.9679 73.4325 76.3723 70.2404 85.0319 70.1936C93.688 70.1459 96.8455 73.2964 101.053 76.3334C105.219 79.3436 108.754 82.1839 116.053 82.1458M44.1132 50.268C44.7907 27.6025 63.3508 9.54075 85.5682 9.92893C107.787 10.3119 125.245 29.0002 124.569 51.6647C123.891 74.331 105.332 92.3928 83.1126 92.0064C60.8952 91.6208 43.4312 72.9351 44.1132 50.268ZM62.7919 50.592C63.155 38.4518 73.0987 28.7775 84.999 28.9846C96.8975 29.1917 106.249 39.2004 105.886 51.3424C105.526 63.4801 95.5847 73.1561 83.6828 72.9498C71.7816 72.7419 62.428 62.7323 62.7919 50.592ZM95.9062 4.41378C96.7415 6.77927 99.0819 7.50538 101.105 6.02283C103.13 4.54028 105.047 5.35737 105.366 7.84244C105.681 10.3214 107.823 11.5111 110.121 10.4765C112.42 9.44717 114.125 10.6386 113.915 13.1254C113.699 15.613 115.551 17.21 118.019 16.6745C120.488 16.139 121.91 17.6501 121.177 20.0373C120.443 22.4227 121.92 24.3593 124.454 24.3454C126.989 24.3273 128.061 26.094 126.841 28.2732C125.62 30.4524 126.656 32.6446 129.142 33.1472C131.63 33.648 132.31 35.5958 130.656 37.4752C128.999 39.352 129.551 41.7019 131.887 42.7036C134.216 43.7035 134.472 45.7432 132.453 47.2335C130.431 48.7265 130.481 51.1379 132.553 52.5936C134.628 54.0476 134.449 56.0907 132.158 57.1322C129.863 58.1772 129.399 60.5384 131.128 62.384C132.854 64.2296 132.246 66.1895 129.781 66.7337C127.311 67.2813 126.361 69.4934 127.663 71.6475C128.966 73.8042 127.956 75.5917 125.425 75.6186C122.891 75.6481 121.488 77.6106 122.313 79.9822C123.135 82.3529 121.772 83.8926 119.285 83.3979C116.794 82.9057 115.008 84.5347 115.314 87.0189C115.62 89.5022 113.958 90.724 111.621 89.731C109.286 88.7397 107.19 89.9649 106.968 92.4552C106.744 94.9403 104.859 95.792 102.777 94.3476C100.698 92.8988 98.3843 93.6674 97.64 96.0468C96.8983 98.4322 94.8699 98.8767 93.1352 97.0389C91.4005 95.1985 88.9709 95.4731 87.7422 97.648C86.5101 99.816 84.4305 99.8376 83.1152 97.6879C81.806 95.5364 79.3668 95.3068 77.7049 97.174C76.0387 99.0413 73.9955 98.6332 73.1619 96.2668C72.3284 93.8988 69.9906 93.1752 67.9631 94.6552C65.939 96.1395 64.0206 95.3198 63.7043 92.839C63.3872 90.3583 61.247 89.1712 58.9482 90.2032C56.6494 91.2361 54.9442 90.0412 55.153 87.5544C55.3679 85.0676 53.5179 83.468 51.0493 84.0044C48.579 84.5425 47.1606 83.0296 47.8927 80.6425C48.6258 78.2588 47.1528 76.3179 44.6183 76.3378C42.0839 76.3534 41.0086 74.5849 42.2277 72.4092C43.4529 70.2308 42.414 68.0325 39.9272 67.53C37.4387 67.0318 36.7602 65.0839 38.4152 63.208C40.0693 61.3295 39.5173 58.977 37.1865 57.9779C34.8513 56.9771 34.5992 54.9374 36.6155 53.4462C38.637 51.9576 38.5893 49.5462 36.5141 48.0888C34.4389 46.6331 34.6209 44.5899 36.9153 43.5484C39.2071 42.506 39.6707 40.1431 37.9438 38.2958C36.2169 36.4528 36.8226 34.4937 39.2912 33.9461C41.7589 33.4002 42.7121 31.1898 41.4106 29.0314C40.1048 26.8782 41.1108 25.0889 43.6453 25.0638C46.1806 25.0343 47.5808 23.0726 46.7577 20.6984C45.9354 18.3277 47.294 16.7906 49.7886 17.2828C52.2763 17.7741 54.0604 16.1468 53.7562 13.6626C53.4512 11.1793 55.1131 9.95665 57.4492 10.9479C59.7852 11.9383 61.8821 10.7148 62.1004 8.22889C62.3266 5.73776 64.2138 4.88947 66.2933 6.33563C68.3764 7.78092 70.6864 7.01495 71.4281 4.63473C72.1759 2.24931 74.2 1.80567 75.9329 3.64434C77.6694 5.48128 80.0973 5.20921 81.3268 3.03434C82.5598 0.862935 84.6393 0.846474 85.9547 2.99535C87.2674 5.14509 89.7031 5.37384 91.3676 3.51004C93.0304 1.63584 95.0735 2.04568 95.9062 4.41378Z"
                            stroke="currentColor" stroke-miterlimit="10" />
                    </svg>
                </div>
                <hr style="border: 0.5px #bbb dashed; margin-bottom: 0.6rem;">
                <div id="content" class="content" style="column-count: 2;" :class="{ active : isActive }" v-html="blogText">
                </div>

            </div>
            <div class="cardBack" :class="{ active: isActive && !isIdle }">
            </div>
        </div>
    </div>
</template>

<style scoped>
@import url('https://fonts.cdnfonts.com/css/jailbird-jenna');
@import url('https://fonts.cdnfonts.com/css/shoplifter');

.contentWrapper {
    position: absolute;
    width: 100%;
    height: 100%;
    padding: 1.2rem;
    color: var(--color-3);
    translate: 0 0 2rem;
    background-color: var(--color-4);
    overflow: hidden;
    font-size: 1rem;
    opacity: 0;
    transition: all 0.05s;
    transition-delay: 50ms;
    /* display: grid; */
    /* grid-template-columns: 1fr 1fr; */
    /* column-gap: 1.5rem; */
}

.contentWrapper.active {
    opacity: 1;
    transition: all 0.2s;
    transition-delay: 300ms;
}

.contentWrapper h1 {
    font-family: 'Shoplifter', sans-serif;
    font-size: 2rem;
}

.content :deep(p) {
    font-family: 'Courier New', Courier, monospace;
    font-size: 65%;
    text-indent: 1rem;
    margin-bottom: 0.4rem;
    text-align: justify;
    break-inside: avoid;
}

.content :deep(p strong) {
    font-weight: bold;
}

.content :deep(h1),
.content :deep(h2),
.content :deep(h3),
.content :deep(h4) {
    font-family: 'Courier New', Courier, monospace;
    font-size: 80%;
    font-weight: bold;
    margin-bottom: 0.4rem;
    text-align: left;
}

.content :deep(h5) {
    font-family: 'Courier New', Courier, monospace;
    font-size: 70%;
    line-height: 0.5rem;
    /* font-weight: bold; */
    margin-bottom: 0.4rem;
    text-align: right;
    opacity: 60%;
}

.content :deep(a) {
    font-family: 'jailbIrD JenNA', sans-serif;
    color: var(--color-3);
    font-size: 80%;
    opacity: 0.8;
    /* text-transform: none; */
    /* font-style: italic; */
    text-decoration: none;
}

#content.active :deep(a) { 
    pointer-events: all;
}

h1.title {
    font-weight: bold;
    font-size: 3rem;
    font-family: 'jailbIrD JenNA', sans-serif;
    margin: 0.6rem;
    padding: 0.5rem;
    color: var(--color-5);
}

h2.date {
    font-weight: bold;
    font-size: 2rem;
    font-family: 'jailbIrD JenNA', sans-serif;
    line-height: 2.5rem;
    margin: 0.6rem;
    text-align: right;
    opacity: 0.4;
}

h1.idle,
h2.idle {
    -webkit-text-stroke: 1.5px #606060c6;
    color: #00000000;
    transition: color 0.05s ease;
}

div.headerInfo {
    display: grid;
    grid-template-columns: 3fr 1fr;
}

p.description {
    font-family: 'Courier New', Courier, monospace;
    margin: 0.6rem;
    opacity: 0.8;
    padding: 0.7rem;
}

p.description.idle {
    color: var(--color-3);
}

strong {
    font-weight: bold;
}

.postalIcon {
    position: absolute;
    left: 95%;
    top: 2.9rem;
    /* padding: 0.1rem; */
    transform: translateX(-100%) translateY(-100%);
    opacity: 0.2;
    color: var(--color-5);
}

.postalIcon.idle {
    opacity: 0.08;
    color: var(--color-2);
}

.cardContainer {
    position: relative;
    perspective: 1000px;
    padding: 2rem;
    rotate: 4deg;
    max-width: 39rem;
    /* filter:invert(); */
    transition: all 0.1s;
}

.cardContainer.active {
    perspective: 500px;
    rotate: 0deg;
    transition: all 0.1s;
}

@keyframes cardAnim {
    0% {
        /* left: 0%; */
        rotate: 0;
    }

    50% {
        /* left : -50%; */
        rotate: -10deg;
    }

    100% {
        /* left: 0%; */
        rotate: 0;
    }
}

.card {
    display: grid;
    /* grid-template-columns: 5fr 5fr 1fr; */
    position: relative;
    padding: 1rem;
    font-size: 100%;
    color: #eee;
    transition: transform 0.4s cubic-bezier(.13, 1.94, .32, .67);
    transform-style: preserve-3d;
}

.card.active {
    display: grid;
    /* grid-template-columns: 5fr 5fr 1fr; */
    position: relative;
    padding: 1rem;
    font-size: 100%;
    color: #eee;
    /* left: -20%; */
    animation-name: cardAnim;
    animation-duration: 250ms;
    animation-delay: 150ms;
    animation-timing-function: cubic-bezier(1, 0, 0, 1);
    animation-iteration-count: 1;
    /* left: -50%; */
    /* transition: left 0.2s cubic-bezier(.13, 1.94, .32, .67); */
    /* transform-style: preserve-3d; */
    /* translate: -20% 0; */
    /* filter : invert(); */
}

.card * {
    pointer-events: none;
}

.card::before {
    position: absolute;
    content: '';
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: var(--color-2);
    z-index: -1;
    transition: all 0.05s;
    outline: #fefefe 3px solid;
}

.card::after {
    position: absolute;
    content: '';
    top: 1rem;
    left: 1rem;
    width: 100%;
    height: 100%;
    background-color: var(--color-1);
    z-index: -2;
    transform: rotate(6deg) translateZ(-4rem);
    transition: all 0.05s;
}


@keyframes paperAnim {
    0% {
        transform: translateZ(-8rem);
        background-color: var(--color-1);
    }

    10% {
        transform: rotate(15deg) translateZ(-8rem) translateX(15%);
        background-color: var(--color-1);
    }

    40% {
        content: '';
        transform: rotate(10deg) rotateY(-15deg) translateZ(3rem) translateX(110%);
        background-color: var(--color-1);
    }

    90% {
        transform: rotate(0deg) translateZ(2rem);
        background-color: var(--color-4);
    }
}

.card.active::after {
    position: absolute;
    content: '';
    font-family: 'Courier New', Courier, monospace;
    font-size: 1.2rem;
    color: var(--color-3);
    padding: 2rem;
    top: 0rem;
    left: 0rem;
    width: 100%;
    height: 100%;
    background-color: var(--color-4);
    z-index: -2;
    transform: rotate(0deg) translateZ(2rem);
    /* transition: all 0.05s; */
    animation-name: paperAnim;
    animation-duration: 0.35s;
    animation-timing-function: cubic-bezier(1, 0, 0, 1);
    animation-iteration-count: 1;
    outline: var(--color-5) 1.5px solid;
}

.card.idle::before {
    position: absolute;
    content: '';
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: #8f8f8f37;
    outline: #aaaaaa 1px solid;
    backdrop-filter: blur(10rem);
    z-index: -1;
    transition: all 0.05s;
}

.card.idle::after {
    position: absolute;
    content: '';
    top: 1rem;
    left: 1rem;
    width: 100%;
    height: 100%;
    background-color: transparent;
    outline: #eaeaea 1px solid;
    z-index: -2;
    transform: rotate(6deg) translateZ(-1rem);
}

.indicator {
    position: absolute;
    width: 3rem;
    height: 3rem;
    z-index: 100;
    top: 68%;
    left: 100%;
    rotate: -85deg;
    translate: 0 0 0.4rem;
    /* transform: rotateX(-5deg) rotateY(5deg); */
    /* outline: var(--color-2) 5px solid; */
    /* scale: 1; */
    color: var(--color-1);
    transition: all 0.4s cubic-bezier(.13, 1.94, .32, .67);
    transition-delay: 100ms;
    transform: rotateY(30deg) translateX(-50%) translateY(-50%);

}

.indicator.idle {
    top: 68%;
    left: 91%;
    color: var(--color-5);
    translate: 0 0 0.4rem;
    rotate: -85deg;
    transition: all 0.1s ease;
    transform: rotateY(30deg) translateX(-50%) translateY(-50%);
}

.indicator.active {
    position: absolute;
    width: 3rem;
    height: 3rem;
    z-index: 100;
    top: 68%;
    left: 95%;
    color: #aaa;
    rotate: -85deg;
    translate: 0 0 0.4rem;
    /* color: var(--color-5); */
    transition: all 0.4s cubic-bezier(.13, 1.94, .32, .67);
    transform: rotateY(30deg) translateX(-50%) translateY(-50%);
}

@keyframes fx {
    0% {
        top: 0%;
        scale: 1 0;
    }

    25% {
        scale: 1;
    }

    100% {
        top: 120%;
        scale: 1 0;
    }
}

.indicator::before {
    position: absolute;
    content: '';
    scale: 0;
    width: 4rem;
    height: 4rem;
    background: var(--color-1);
    color: white;
    z-index: 0;
    animation-name: fx;
    animation-duration: 150ms;
    animation-timing-function: ease;
    translate: 1rem 1rem 1rem;
    animation-delay: 100ms;
}

.indicator.idle::before {
    scale: 1;
    position: absolute;
    content: '';
    width: 4rem;
    height: 4rem;
    background: transparent;
    color: white;
    z-index: -10;
    translate: 1rem 1rem 1rem;
    animation-name: none;
}

.indicator.active::before {
    opacity: 0;
}

.cardBack {
    position: absolute;
    width: 100%;
    height: 95%;
    transform-origin: 0 0 0;
    translate: 0 0 -1rem;
    transform: rotateZ(0deg);
    background-color: transparent;
}

.cardBack.active {
    translate: -0.2rem 0 1.5rem;
    transform: rotateZ(4deg);
    background-color: var(--color-4);
    transition: all 0.2s;
    transition-delay: 200ms;
}
</style>