<template>

  <div @wheel="scroll" @input="emitValue" class="slidecontainer">
    <input @mousedown="emitValue" ref="slider" type="range" min="1" max="100" class="slider" id="myRange" @changeValue="(val)=>setVolume(val/100)">
  </div>

</template>
<script setup lang="ts">

// RANGE SLIDER

import {ref, onMounted,} from 'vue'

const emit = defineEmits(['changeValue'])
let props = defineProps(["value"])

let slider = ref()

function emitValue() {
  emit("changeValue", slider.value.value === "1" ? 0 : slider.value.value)
}

function scroll(e: WheelEvent) {
  slider.value.value = parseInt(slider.value.value) + (e.deltaY < 0 ? 10 : -10)
  emitValue()
}

onMounted(() => {
  slider.value.value = props.value
  emitValue()
})

function setValue(value: number) {
  slider.value.value = value
  emitValue()
}

function getValue(){
  return slider.value.value
}

defineExpose({setValue,getValue})
</script>

<style scoped>
div.slidecontainer {

}

div.slidecontainer {
  float: left;
  width: 30px;
}

input#myRange {
  -webkit-appearance: slider-vertical;
  float: left;
  width: 20px;
  height: 200px;
}
</style>