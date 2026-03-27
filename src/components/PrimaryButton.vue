<template>
  <el-button type="primary" @click="handleClick">
    <slot>恶趣味请问11大大我的</slot>
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