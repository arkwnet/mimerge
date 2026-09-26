<template>
  <div class="checkbox">
    <label class="switch">
      <input
        type="checkbox"
        :checked="value"
        @input="emit('update:value', $event.target.checked)"
      />
      <div class="slider"><div class="circle"></div></div>
    </label>
    <div class="label">{{ label }}</div>
  </div>
</template>

<script setup lang="js">
import { defineProps } from 'vue'
const props = defineProps({
  label: String,
  value: Boolean,
})
const emit = defineEmits(['update:value'])
</script>

<style scoped lang="css">
.checkbox {
  margin-right: 18px;
  float: left;
  overflow: hidden;
}

.checkbox .switch .slider {
  width: 48px;
  height: 28px;
  position: relative;
  background-color: #bdbdbd;
  border-radius: 14px;
  display: block;
  user-select: none;
  float: left;
  transition: background-color 0.1s ease;
  cursor: pointer;
}

.checkbox .switch .circle {
  width: 20px;
  height: 20px;
  position: absolute;
  left: 5px;
  top: 4px;
  background-color: #fff;
  border-radius: 10px;
  transition: left 0.1s ease;
}

.checkbox .switch input[type='checkbox'] {
  display: none;
}

.checkbox .switch input[type='checkbox']:checked + .slider {
  background-color: #4caf50;
}

.checkbox .switch input[type='checkbox']:checked + .slider .circle {
  left: 23px;
}

.checkbox .label {
  margin-left: 8px;
  padding: 2px;
  float: left;
  color: #424242;
}

@media screen and (max-width: 700px) {
  .checkbox {
    width: 100%;
    margin-bottom: 10px;
    overflow: hidden;
  }
}
</style>
