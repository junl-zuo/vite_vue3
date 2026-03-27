<template>
  <el-button type="danger" @click="handleClick">
    <slot>Danger Button</slot>
  </el-button>
</template>

<script setup>
const handleClick = () => {
  console.log('Danger button clicked');
};
</script>

<style scoped>
/* Add any custom styles here */
</style>