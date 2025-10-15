<script setup>
import BinauralBeatGenerator from "./components/BinauralBeatGenerator.vue"
</script>

<template>
  <div id="app">
    <BinauralBeatGenerator />
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  background-color: #f5f5f5;
}

#app {
  min-height: 100vh;
  padding: 20px;
}

@media (max-width: 600px) {
  #app {
    padding: 0;
  }
}
</style>
