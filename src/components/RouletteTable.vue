<script setup lang="ts">
/* eslint-disable prettier/prettier */
import {reactive, ref, computed, onMounted} from "vue";

let tableNumberRows = [
  [3, 6, 9, 12, 15, 18, 21, 24, 27, 30, 33, 36],
  [2, 5, 8, 11, 14, 17, 20, 23, 26, 29, 32, 35],
  [1, 4, 7, 10, 13, 16, 19, 22, 25, 28, 31, 34]]

let emit = defineEmits(["bet-placed"])
let currentPosition:{}
let placedchip = ref()

let rouletteWheel = ref()
let chip = ref()
let table = ref()
let previousX = 0
let currentX = 0
let previousY = 0
let currentY = 0
let currentlyDrawingLine = false;
let leftCanvasWhileDrawing = false
let anythingDrawn = false

let chipIsPlaced = false
let chipPlacedPosition = {top: 0, left: 0}
let chipHeight = 15
let chipWidth = 15
let borderZoneWidth = 10
let numberCellWidth = 39.6
let numberCellHeight = 55

function findChipPosition(left: number, top: number) {
// make sure cursor is on any number field

  let newPosition = {left: -100, top: -100, row: -1, column: -1, numbers: [-1]}
  if (left > 130 && left < 593 && top > 65 && top < 218) { // real left:122- 597, top:56- 222
    console.log("on numbers")

    // get left position
    let cellX = (Math.floor(left / numberCellWidth))
    let remainderX = left % numberCellWidth
    let onVerticalBar = remainderX < borderZoneWidth
    console.log("Remainder X: " + remainderX)
    newPosition.left = cellX * numberCellWidth + (onVerticalBar ? 0 : chipWidth)

    // get top position
    let cellY = (Math.floor(top / numberCellHeight))
    let remainderY = top % numberCellHeight
    let onHorizontalBar = remainderY < borderZoneWidth
    console.log("Remainder Y: " + remainderY)

    let onCross = onHorizontalBar && onVerticalBar
    if (onCross) {
      newPosition.left = cellX * numberCellWidth + (onVerticalBar ? 0 : onHorizontalBar ? numberCellWidth / 2 : chipWidth)
      newPosition.top = cellY * numberCellHeight + (onHorizontalBar ? 0 : numberCellHeight - chipHeight / 2)
    } else {
      // get left position
      newPosition.left = cellX * numberCellWidth
      if (onVerticalBar) {

      } else {

        if (onHorizontalBar) {
          newPosition.left += numberCellWidth / 2
        } else {
          newPosition.left += chipWidth
        }
      }

      // get top position

      newPosition.top = cellY * numberCellHeight
      if (onHorizontalBar) {

      } else {
        if (onVerticalBar) {
          newPosition.top += numberCellHeight / 2

        } else {
          newPosition.top += numberCellHeight - chipHeight / 1.5

        }
      }
    }
    // get row and column number
    // subtract part of image thats not numbers
    cellY -= 1
    cellX -= 3
    console.log(cellX)
    console.log(cellY)
    newPosition.numbers = [tableNumberRows[cellY][cellX]]

    if (onHorizontalBar) {
      newPosition.numbers.push(tableNumberRows[cellY - 1][cellX])
    }
    if (onVerticalBar) {
      newPosition.numbers.push(tableNumberRows[cellY][cellX - 1])
    }
    if (onCross) {
      newPosition.numbers.push(tableNumberRows[cellY - 1][cellX - 1])
    }
    console.log(newPosition.numbers)
    currentPosition=newPosition

    //newPosition.numbers=[tableNumberRows[cellX][cellY]]


  }
  return newPosition
}


onMounted(() => {

})

function processDrawEvent(res: string, e: MouseEvent) {
  previousX = currentX;
  previousY = currentY;
  currentX = e.clientX - table.value.getBoundingClientRect().x
  currentY = e.clientY - table.value.getBoundingClientRect().y
  if (res === 'down') {
    currentlyDrawingLine = true;
    let shouldDrawDot = true;

  }
  if (res === 'click') {
    chipIsPlaced = true
    currentlyDrawingLine = true;
    let shouldDrawDot = true;
    chipPlacedPosition = {top: chip.value.style.top, left: chip.value.style.left}
    placedchip.value.style.display = "block"
    placedchip.value.style.top = chip.value.style.top
    placedchip.value.style.left = chip.value.style.left
    emit("bet-placed", currentPosition.numbers)

  }

  if (res === 'move') {
    console.log("move")
    if (currentlyDrawingLine) {
    }
    console.log(`X:${currentX}, Y:${currentY}`)
    let chipPosition = findChipPosition(currentX, currentY)
    console.log(chipPosition)
    if (chipPosition.left === -100) {
      chip.value.style.display = "none"
    } else {
      chip.value.style.display = "block"
      chip.value.style.top = chipPosition.top + "px"
      chip.value.style.left = chipPosition.left + "px"

    }
  }
}

function consolelog(m: string) {
  console.log(m)
}
</script>

<template>
  <div id="all">
    <div ref="table" id="table_and_chip"
         @click="(e)=>{processDrawEvent('click', e)}"
         @mouseenter="(e)=>{processDrawEvent('enter', e)}"
         @mousemove="(e)=>{processDrawEvent('move', e)}"
         @mouseleave="(e)=>{processDrawEvent('leave', e)}"
         @mousedown="(e)=>{processDrawEvent('down', e)}">
      <div id="table_div"><img id="table" src="/roulette-table.jpg"/></div>
      <div id="chip" class="chip" ref="chip"><img src="/chip.png"/></div>
      <div id="placedchip" class="chip" ref="placedchip"><img src="/chip.png"/></div>

    </div>
  </div>
</template>

<style lang="scss">

#all {
  max-width: 1480px;
  margin: 0 auto;
  padding: 3rem;
  font-weight: normal;
}

#table_and_chip {
  position: relative;
  height: 360px;
  width: 720px;
  cursor:pointer;

}

img#table {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -2;
}


div#placedchip {
  display: none;
}

div.chip {
  transition: top 0.2s ease,
  left 0.2s ease;
  position: absolute;
  border-radius: 10px;
  height: 15px;
  width: 15px;
  top: 20px;
  left: 20px;
  display:none;

  img {
    position: absolute;
    top: -50%;
    left: -50%;
    height: 100%;
    width: 100%;
  }
}


body {
  background-color: white;
}

</style>
