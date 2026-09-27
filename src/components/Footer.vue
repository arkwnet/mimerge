<template>
  <div class="footer">
    <p><b>MiMerge</b> - Version {{ version }}</p>
    <p>
      Copyright (c) 2026 <a href="https://arkw.net/">Sora Arakawa</a>. Licensed under the MIT
      License.
    </p>
    <p><a href="https://github.com/arkwnet/mimerge">GitHub Repository</a></p>
  </div>
</template>

<script setup lang="js">
import { ref, onMounted } from 'vue'
const version = ref('')

onMounted(() => {
  version.value = import.meta.env.PACKAGE_VERSION
})
</script>

<style scoped lang="scss">
@use '../assets/sass/color';

.footer {
  padding: 20px;
  color: color.$grey-600;

  a {
    color: color.$grey-600;

    &:hover {
      color: color.$grey-500;
    }
  }
}
</style>
