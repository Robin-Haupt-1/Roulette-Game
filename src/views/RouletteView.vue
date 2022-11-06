<script setup lang="ts">
/* eslint-disable prettier/prettier */
import {reactive, ref, computed, onMounted} from "vue";


</script>
<template>
  <div>
    <div class="result"><span class="test">sdfsd</span></div>

    <h2>Roulette Game</h2>
    <h3></h3>
    <div>
      <div class="roulette-wheel">
        <div class="layer-2 wheel" style="transform: rotate(0deg);"></div>
        <div class="layer-3"></div>
        <div class="layer-4 wheel" style="transform: rotate(0deg);"></div>
        <div class="layer-5"></div>
        <div class="ball-container" style="transform: rotate(0deg);">
          <div
              class="ball"
              style="transform: translate(0, -163.221px);"
          ></div>
        </div>
        <svg width="380" height="380">
          <circle
              cx="190"
              cy="190"
              r="190"
              style="touch-action: none;"
          ></circle>
        </svg>
      </div>
    </div>
  </div>
</template>

<style lang="scss">

body{
  background-color:white;
}

$title: 'This is title';
.post__title {
  content: quote($title);
}

@function double($variable) {
  @return $variable *6;
}

.post {
  font-size: double(2px);
}

.result {
  height: 100px;
  width: 100px;

  .test {
    color: green;
    font-size: double(10px);
  }
}
</style>
