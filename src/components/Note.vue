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

<style scoped lang="scss">
.note {
  margin-bottom: 2px;
  padding: 10px 15px;
  overflow: hidden;
  background-color: #fff;

  .avatar {
    width: 70px;
    float: left;

    img {
      width: 50px;
      height: 50px;
      margin-left: 0px;
      border-radius: 25px;
    }
  }

  .user {
    width: 400px;
    float: left;

    .name {
      padding-top: 3px;
      color: #212121;
      font-size: 18px;
      font-weight: bold;
      overflow-wrap: break-word;
      word-wrap: break-word;
    }

    .userid {
      margin-top: -4px;
      color: #616161;
    }
  }

  .information {
    margin-left: 20px;
    padding-top: 12px;
    float: left;
    overflow: hidden;

    .name {
      margin-right: 6px;
      padding: 2px;
      float: left;
    }

    .chip {
      height: 28px;
      margin: 0 4px;
      padding: 2px 14px;
      color: #fff;
      background-color: #f44336;
      border-radius: 6px;
      float: left;
    }

    .following {
      background-color: #f44336;
    }

    .followers {
      background-color: #2196f3;
    }

    .disabled {
      opacity: 0.3;
    }
  }
}
</style>
