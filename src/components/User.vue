<template>
  <div class="user">
    <div class="profile">
      <a :href="url" target="blank">
        <div class="avatar">
          <img :src="avatar" />
        </div>
        <div class="text">
          <div class="name">{{ name }}</div>
          <div class="userid">{{ id }}</div>
        </div>
      </a>
    </div>
    <div class="information">
      <component
        :is="value[0].url ? 'a' : 'div'"
        :href="value[0].url || undefined"
        :target="value[0].url ? 'blank' : undefined"
      >
        <div class="name">アカウントA</div>
        <div class="chip following" :class="{ disabled: value[0].following === false }">
          フォロー
        </div>
        <div class="chip followers" :class="{ disabled: value[0].followers === false }">
          フォロワー
        </div>
      </component>
    </div>
    <div class="information">
      <component
        :is="value[1].url ? 'a' : 'div'"
        :href="value[1].url || undefined"
        :target="value[1].url ? 'blank' : undefined"
      >
        <div class="name">アカウントB</div>
        <div class="chip following" :class="{ disabled: value[1].following === false }">
          フォロー
        </div>
        <div class="chip followers" :class="{ disabled: value[1].followers === false }">
          フォロワー
        </div>
      </component>
    </div>
  </div>
</template>

<script setup lang="js">
import { defineProps } from 'vue'
const props = defineProps(['avatar', 'name', 'id', 'url', 'value'])
</script>

<style src="../assets/sass/components/User.scss" lang="scss" scoped></style>
