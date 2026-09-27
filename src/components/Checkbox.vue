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

<style src="../assets/sass/components/Checkbox.scss" lang="scss" scoped></style>
