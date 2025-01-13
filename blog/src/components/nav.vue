<script setup>
import { ref, computed } from 'vue';
const activeLink = ref('home');
const emit = defineEmits(['navItem']);
const setActive = (link) => {
    if (!['home', 'blogs', 'about'].includes(link))
        link = 'home';
    emit('navItem', link);
    activeLink.value = link;
}
const homeStyle = computed(() => {
    if (activeLink.value == 'home') {
        return {
            transform: "scale(1.3) rotate3d(1.65, .65, 0.1, 45deg)"
        }
    }
    return { transform: "scale(0.7) rotateZ(-8deg)" }
});
const blogsStyle = computed(() => {
    if (activeLink.value == 'blogs') {
        return {
            transform: "scale(1.3) rotate3d(-.55, .15, 0.1, -25deg)"
        }
    }
    return { transform: "scale(0.7) rotateZ(0deg)" }
});
const aboutStyle = computed(() => {
    if (activeLink.value == 'about') {
        return {
            transform: "scale(1.3) rotate3d(-.55, .35, 0.1, 45deg)"
        }
    }
    return { transform: "scale(0.7) rotateZ(5deg)" }
});
const hash = window.location.hash;
setActive(hash.slice(1));

const indicatorStyle = computed(() => {
    switch (activeLink.value) {
        case 'home':
            return { transform: "translateX(-50%) translate(-7.7rem, -0.55rem) rotate3d(1, -1, 1, 65deg)" };
        case 'blogs':
            return { transform: "translateX(-50%) translate(1.0rem, -0.6rem) rotate3d(6, 1, 3, -45deg)" };
        case 'about':
            return { transform: "translateX(-50%) scale(1.1) translate(6.5rem, -.25rem) rotate3d(2, -1, -1, -55deg)" };
    }
})
</script>

<template>
    <main>
        <div class="container">
            <nav class="nav">
                <div id="indicator" :style="indicatorStyle"></div>
                <div class="nav-links">
                    <a href="#home" @click="setActive('home')" :class="{ active: activeLink === 'home' }"
                        :style="homeStyle"><span class="strong">H</span>ome</a>
                    <a href="#" @click="setActive('blogs')" :class="{ active: activeLink === 'blogs' }"
                        :style="blogsStyle"><span class="strong">B</span>logs</a>
                    <a href="#about" @click="setActive('about')" :class="{ active: activeLink === 'about' }"
                        :style="aboutStyle"><span class="strong">A</span>bout</a>
                </div>
            </nav>
        </div>
    </main>
</template>

<style scoped>
@import url('https://fonts.cdnfonts.com/css/president-nixon');
@import url('https://fonts.cdnfonts.com/css/mixpunchout');

#indicator {
    position: absolute;
    background-color: var(--color-1);
    width: 5rem;
    height: 2.6rem;
    top: 1.5rem;
    left: 50%;
    color: var(color-2);
    transition: all 0.1s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.nav {
    position: fixed;
    top: 0;
    left: 50%;
    width: 100%;
    height: 4.4rem;
    margin-top: 1rem;
    transform: translateX(-50%);
    /* padding: .2rem; */
    z-index: 1000;
    transition: all 0.3s ease;
}

.nav::after {
    position: absolute;
    content: '';
    top: -4rem;
    width: 120%;
    height: 100%;
    background-color: var(--color-5);
    filter: blur(25px);
    z-index: -1;
}

.nav-links {
    display: flex;
    justify-content: center;
    gap: 1rem;
}

a {
    line-height: 2.8rem;
    text-decoration: none;
    text-align: center;
    text-transform: uppercase;
    color: var(--color-2);
    font-weight: bold;
    font-family: 'MixPunchOut', sans-serif;
    font-style: italic;
    font-size: 1.0rem;
    padding: 0.15rem 1rem;
    transform: scale(0.7);
    transition: all 0.15s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

a:hover {
    color: #702222;
}

a.active {
    color: #eaeaea;
    transform: scale(1.2);
    position: relative;
}

a.active::before {
    content: "";
    position: absolute;
    left: 15%;
    right: 0;
    top: 14%;
    height: 70%;
    width: 80%;
    background: #333;
    z-index: -1;
}

.strong {
    font-size: 2rem;
    font-weight: bolder;
    font-family: 'President Nixon', sans-serif;
    font-style: inherit;
}
</style>