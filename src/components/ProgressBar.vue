<template>
    <div class="progress-bar">
      <div class="progress" :style="{ width: value + '%' }"></div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      value: {
        type: Number,
        required: true,
      },
    },
  };
  </script>
  
  <style scoped>
  .progress-bar {
    width: 100%;
    background-color: #e0e0e0;
    border-radius: 5px;
    height: 10px;
    overflow: hidden;
  }
  
  .progress {
    height: 100%;
    background-color: #7f7fd5;
    transition: width 0.3s ease;
  }
  </style>
  