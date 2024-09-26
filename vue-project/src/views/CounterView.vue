<template>
  <div class="about">
    <h1>This is my Counter page</h1>
    <button onclick="counter()"></button>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>

<script setup lang="ts">
let number = document.getElementsByTagName('button');
let index = 0;
function counter(){
  index ++;
}
</script>