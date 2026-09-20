<template>
  <Header />
  <Main />
  <Footer />
</template>

<script setup>
import { onMounted } from 'vue'
import { pageview } from 'vue-gtag'
import Header from './components/Header.vue'
import Main from './components/Main.vue'
import Footer from './components/Footer.vue'

onMounted(() => {
  pageview('/')
})
</script>
