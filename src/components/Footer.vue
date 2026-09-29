<template>
  <div class="footer">
    <div class="advertisement">広告</div>
    <Adsense
      adStyle="display:block"
      :clientId="adClient"
      :slotId="adSlot"
      format="auto"
      fullWidthResponsive="true"
    ></Adsense>
    <div class="information">
      <p><b>MiMerge</b> - Version {{ version }}</p>
      <p>
        Copyright (c) 2026 <a href="https://arkw.net/">Sora Arakawa</a>. Licensed under the MIT
        License.
      </p>
      <p><a href="https://github.com/arkwnet/mimerge">GitHub Repository</a></p>
    </div>
  </div>
</template>

<script setup lang="js">
import { ref, onMounted } from 'vue'
import { Adsense } from 'vue3-google-adsense'
const adClient = import.meta.env.VITE_AD_CLIENT
const adSlot = import.meta.env.VITE_AD_SLOT
const version = ref('')

onMounted(() => {
  version.value = import.meta.env.PACKAGE_VERSION
})
</script>

<style src="../assets/sass/components/Footer.scss" lang="scss" scoped></style>
