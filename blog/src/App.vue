<script setup lang="ts">
import Nav from './components/nav.vue';
import Card from './components/card.vue';
import { computed, ref, watch } from 'vue';
// import Ind from './components/mindicator.vue'
const menuItem = ref("home");
let cards = ['home'];
</script>

<template>
    <main>
        <!-- <Ind /> -->
        <div class="wrapper">
            <Nav @navItem="(item : string) => {menuItem = item;}"/>
            <Card :src = "menuItem"/>
        </div>
    </main>
</template>
<style>
.wrapper {
    display: flex;
    flex-direction: column;
    place-items: center;
    margin-top: 10%;
    /* justify-content: center; */
    /* align-items: center; */
    /* height: 100vh; */
    z-index: 100;
}
</style>
