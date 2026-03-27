<template>
  <el-button type="primary" @click="handleClick">
    <slot>Primary Button</slot>
  </el-button>
</template>

<script setup>
const handleClick = () => {
  console.log('Primary button clicked');
};
</script>

<style scoped>
/* Add any custom styles here */
</style>