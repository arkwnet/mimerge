<template>
  <div class="note">
    <div class="avatar">
      <img :src="avatarUrl" />
    </div>
    <div class="user">
      <div class="name">{{ name }}</div>
      <div class="userid">{{ userid }}</div>
    </div>
    <div class="information">
      <div class="name">アカウントA</div>
      <div class="chip following" :class="{ disabled: value[0].following === false }">フォロー</div>
      <div class="chip followers" :class="{ disabled: value[0].followers === false }">
        フォロワー
      </div>
    </div>
    <div class="information">
      <div class="name">アカウントB</div>
      <div class="chip following" :class="{ disabled: value[1].following === false }">フォロー</div>
      <div class="chip followers" :class="{ disabled: value[1].followers === false }">
        フォロワー
      </div>
    </div>
  </div>
</template>

<script setup lang="js">
import { defineProps } from 'vue'
const props = defineProps(['avatarUrl', 'name', 'userid', 'value'])
</script>

<style src="../assets/sass/components/Note.scss" lang="scss" scoped></style>
