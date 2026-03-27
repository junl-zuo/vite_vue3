<template>
  <el-button type="success" @click="handleClick">
    <slot>Success Button22</slot>
  </el-button>
</template>

<script setup>
const handleClick = () => {
  console.log('Success button clicked');
};
</script>

<style scoped>
/* Add any custom styles here */
</style>