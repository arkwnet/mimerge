<template>
  <div class="header">
    <div class="logo">
      <img src="../assets/img/logo.png" />
    </div>
  </div>
</template>

<style scoped lang="scss">
@use '../assets/sass/color';

.header {
  height: 80px;
  background-color: color.$white;

  .logo {
    padding-top: 10px;
    padding-left: 20px;

    img {
      height: 60px;
    }
  }
}
</style>
