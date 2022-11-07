<script setup lang="ts">
/* eslint-disable prettier/prettier */
import {reactive, ref, computed, onMounted} from "vue";
import RouletteWheel from '../components/RouletteWheel.vue'
import RouletteTable from '../components/RouletteTable.vue'
import ScrollableRangeSlider from "../components/ScrollableRangeSlider.vue"

let betTypePayout = {1: 35, 2: 17, 3: 11, 4: 8}
let currentFunds = ref(10000)
let betTypeNames = {1: "Straight up", 2: "Split", 3: "Street", 4: "Corner"}
let startSliderValue = 10
let rangeSlider = ref()
let rouletteWheel = ref()
let status = ref("place-bet")

let pastBets = ref([])

let betType = ref("")
let betAmount = ref(50)
let betNumbers = ref()
let betNumbersString = ref()
let betPayout = ref()

let currentBet: pastBet

interface Bet {
  type: string
  numbers: {}
}

function onSliderValueChange(newVal: number) {
  console.log(newVal)
  betAmount.value = (newVal ** 1.5).toFixed(2)
  betAmount.value = newVal * 5

}

let bet = ref()

interface pastBet {
  amount: number,
  type: string
  numbers: number[],
  winnings: number,
  won: boolean,
  money: number
}

onMounted(() => {
  console.log(localStorage.getItem("pastBets"))
  pastBets.value = JSON.parse(localStorage.getItem("pastBets")??"[]")
  currentFunds.value = parseFloat(localStorage.getItem("currentFunds")??10000)
  console.log(parseFloat(localStorage.getItem("currentFunds"))+"ff")
})

function consolelog(m: string) {
  console.log(m)
}

function storeBetResult(result: pastBet) {
  let oldBets = pastBets.value
  oldBets.push(result)
  console.log(oldBets)
  pastBets.value = oldBets
  localStorage.setItem("pastBets", JSON.stringify(oldBets))
  localStorage.setItem("currentFunds", String(currentFunds.value))

}

function spinComplete(result: { number: number, color: string }) {
  console.log(result)
  currentBet.won = currentBet.numbers.includes(result.number)
  currentBet.winnings = currentBet.won ? betAmount.value * betTypePayout[currentBet.numbers.length] : -betAmount.value
  currentFunds.value = currentFunds.value + currentBet.winnings
  storeBetResult(currentBet)

}

function doSpin() {
  currentBet = {
    amount: betAmount.value, type: betType.value,
    numbers: betNumbers.value,
  }
  if (betNumbers.value) {
    rouletteWheel.value.doRotate()
  } else {
    alert("Please place the chip first!")
  }
}

function betPlaced(newNumbers: number[]) {
  betNumbers.value = newNumbers
  betNumbersString.value = newNumbers.join(', ')
  betType.value = betTypeNames[newNumbers.length]
  betPayout.value = betTypePayout[newNumbers.length] + " : 1"
}

function clearAllData() {

}

</script>

<template>
  <div id="all">
    <RouletteTable @bet-placed="betPlaced"/>
    <div id="wheel">
      <RouletteWheel ref="rouletteWheel" @resultVisible="spinComplete"/>
    </div>

    <div id="currentfunds">
      <h1 class="flex-center">Current Funds</h1>
      <h2 class="flex-center">{{ currentFunds }} $</h2>
    </div>
    <div id="bet">
      <h1 class="flex-center">Next bet</h1>

      <ScrollableRangeSlider @changeValue="onSliderValueChange" ref="rangeSlider" :value="startSliderValue"/>

      <table>
        <tr>
          <th>Type</th>
          <th>Numbers</th>
          <th>Bet amount</th>
          <th>Payout</th>
        </tr>
        <tr>
          <td>{{ betType }}</td>
          <td>{{ betNumbersString }}</td>
          <td>{{ betAmount }} $</td>
          <td>{{ betPayout }}</td>

        </tr>
      </table>
      <br>
      <div class="flex-center" style="margin-top:20px;">
        <button style="margin:auto" @click="doSpin">Spin the wheel!</button>
      </div>

    </div>
    <div id="pastbets">
      <h1 class="flex-center">Past bets</h1>
      <div id="oldBetsContainer">

        <table>
          <tr>
            <th>Type</th>
            <th>Numbers</th>
            <th>Bet amount</th>
            <th>Payout</th>
            <th>Result</th>
            <th>Winnings</th>
          </tr>
          <tr v-for="bet in pastBets">
            <td>{{ bet.type }}</td>
            <td>{{ bet.numbers.join(', ') }}</td>
            <td>{{ bet.amount }} $</td>
            <td>{{ betTypePayout[bet.numbers.length] }} : 1</td>
            <td>{{ bet.won ? "WON" : "LOST" }}</td>
            <td>{{ bet.winnings }} $</td>
          </tr>
        </table>
      </div>

    </div>
  </div>
</template>

<style lang="scss">
.flex-center {
  display: flex;
  justify-content: center
}

#wheel {
  margin: 20px;
}

th, td {
  padding: 10px;
}

#oldBetsContainer {
  max-height: 400px;
  overflow: auto;
}

#all {
  margin: 0 auto;
  padding: 3rem;
  font-weight: normal;
  display: flex;
  flex-wrap: wrap;
}

button {
  background-color: #4CAF50; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}

#bet {
  background-color: white;
  padding: 20px;
  margin: 10px;

}

#pastbets {
  background-color: white;
  margin: 10px;
  padding: 20px;

}

#currentfunds {
  background-color: white;
  margin: 10px;
  padding: 20px;

}

body {
  background-color: white;
}

</style>
