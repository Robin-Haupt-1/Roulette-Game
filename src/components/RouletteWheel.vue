<script setup>
/* eslint-disable prettier/prettier */
import {reactive, ref, computed, onMounted} from "vue";
import anime from "animejs";
import '../assets/flipper.scss'

import {map, zip, fromEvent, pipe, withLatestFrom} from "@/Observable";

let emit = defineEmits(["resultVisible"])

onMounted(() => {
  window.anime = anime;

  var currentBallRotation = 0;
  var currentWheelRotation = 0;
  var currentWheelIndex = 0;
  var isRotating = false;
  const rouletteWheelNumbers = [0, 32, 15, 19, 4, 21, 2, 25, 17, 34, 6, 27, 13, 36, 11, 30, 8, 23, 10,
    5, 24, 16, 33, 1, 20, 14, 31, 9, 22, 18, 29, 7, 28, 12, 35, 3, 26];

  const getRouletteWheelNumber = index =>
      rouletteWheelNumbers[index >= 0 ? index % 37 : 37 - Math.abs(index % 37)];

  const getRouletteWheelColor = index => {
    const i = index >= 0 ? index % 37 : 37 - Math.abs(index % 37);
    return i == 0 ? "green" : i % 2 == 0 ? "black" : "red";
  };

  window.rouletteWheelNumbers = rouletteWheelNumbers;


  function startRotation(speed) {
    if (isRotating) {
      return;
    }

    isRotating = true;


    const bezier = [0.165, 0.84, 0.44, 1.005];

    const newWheelIndex = currentWheelIndex - speed;
    const result = getRouletteWheelNumber(newWheelIndex);
    const resultColor = getRouletteWheelColor(newWheelIndex);
    (() => {
      const newRotation = currentWheelRotation + (360 / 37) * speed;
      console.log(getRouletteWheelNumber(currentWheelIndex), "---> ", result);
      var myAnimation = anime({
        targets: [".layer-2", ".layer-4"],
        rotate: function () {
          return newRotation;
        },
        duration: function () {
          return 5000;
        },
        loop: 1,
        // easing: "cubicBezier(0.010, 0.990, 0.855, 1.010)",
        easing: `cubicBezier(${bezier.join(",")})`,
        // easing: "cubicBezier(0.000, 1.175, 0.980, 0.990)",
        complete: (...args) => {
          currentWheelRotation = newRotation;
          currentWheelIndex = newWheelIndex;
        }
      });
    })();

    (() => {
      const newRotation = -4 * 360 + currentBallRotation;
      console.log("newRotation", newRotation);
      var myAnimation1 = anime({
        targets: ".ball-container",
        translateY: [
          {value: -3, duration: 0},
          {value: 16, duration: 4000},
          {value: 50, duration: 900}
        ],
        rotate: [{value: newRotation, duration: 4000}],
        duration: function () {
          return 4000; // anime.random(800, 1400);
        },
        loop: 1,
        easing: `cubicBezier(${bezier.join(",")})`,
        complete: () => {
          currentBallRotation = newRotation;
          emit("resultVisible", {number: result, color: resultColor})
          isRotating = false;
        }
      });
    })();
  }

  function offsetEl(el) {
    var rect = el.getBoundingClientRect(),
        scrollLeft = window.pageXOffset || document.documentElement.scrollLeft,
        scrollTop = window.pageYOffset || document.documentElement.scrollTop;
    return {
      top: rect.top + scrollTop,
      left: rect.left + scrollLeft,
      width: rect.width,
      height: rect.height
    };
  }

  function isInBoundaryEl(el, x, y) {
    const o = offsetEl(el);
    return (
        x >= o.left && x <= o.left + o.width && y >= o.top && y <= o.top + o.height
    );
  }

  function isInRadiusEl(el, x, y) {
    const o = offsetEl(el);
    const cx = o.left + o.width / 2;
    const cy = o.top + o.height / 2;
    const dx = x - cx;
    const dy = y - cy;
    const r = o.width / 2;
    return Math.pow(dx, 2) + Math.pow(dy, 2) <= Math.pow(r, 2);
  }

  const documentEvent = eventName =>
      pipe(
          fromEvent(document, eventName),
          map(e =>
              e.type == "touchstart" || e.type == "touchmove"
                  ? {x: e.touches[0].clientX, y: e.touches[0].clientY}
                  : {x: e.clientX, y: e.clientY}
          )
      );


  const tryRotate = ([p0, p1]) => {
    const w = document.querySelector(".layer-2.wheel");
    if (isInRadiusEl(w, p0.x, p0.y)) {
      const d = Math.round(
          Math.sqrt(Math.pow(p0.x - p1.x, 2) + Math.pow(p0.y - p1.y, 2)) / 4
      );
      if (Math.abs(d) > 3) {
        window.startRotation(d);
      }
    }
  };

  //zip(documentEvent("mousedown"))(documentEvent("mouseup")).subscribe({
  //  next: tryRotate
  //});

  zip(documentEvent("touchstart"))(
      pipe(
          withLatestFrom(documentEvent("touchmove"))(fromEvent(document, "touchend")),
          map(([_, r]) => r)
      )
  ).subscribe({
    next: tryRotate
  });

  pipe(
      withLatestFrom(documentEvent("touchmove"))(fromEvent(document, "touchend")),
      map(([_, r]) => r)
  ).subscribe({
    next: e => console.log(e)
  });

  document.querySelector(".roulette-wheel").addEventListener(
      "touchmove",
      e => {
        e.preventDefault();
      },
      {passive: false}
  );

  window.startRotation = startRotation;

})

function doRotate() {
  window.startRotation(Math.floor(Math.random() * 2 * 37))
}

defineExpose({doRotate})
</script>

<template>
  <div>

    <h2></h2>
    <h3></h3>
    <div id="container">

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

body {
  background-color: white;

}

div#container {
  position: relative;
}

.result {
  height: 100px;
  width: 100px;

}


a,
.green {
  text-decoration: none;
  color: hsla(160, 100%, 37%, 1);
  transition: 0.4s;
}


html, body {
  margin: 0;
  padding: 0;
}

body {
  font-family: sans-serif;
  background-image: url(../assets/felt.png), -webkit-radial-gradient(50% 40%, circle farthest-corner, #527c14, #243a0a);
  background-size: 180px 180px, auto;
  background-attachment: fixed;
  user-select: none;
  touch-action: pan-y;
  -webkit-tap-highlight-color: transparent;
  margin: 0;
  padding: 0;
}

#app {
  // overflow: hidden;
  contain: layout style;
  overflow: visible;
}

.roulette-wheel {
  width: 380px;
  height: 380px;
  border-radius: 100%;
  background: url(../assets/roulette_1.jpg);
  background-size: 380px 380px;
  shape-outside: circle(190px);
  margin: 0 0 1em 1em;
  box-shadow: 2px 10px 30px rgba(0, 0, 0, .4);
  position: relative;
  touch-action: none;
  overflow: visible;
}

.roulette-wheel .layer-2, .roulette-wheel .layer-3, .roulette-wheel .layer-4, .roulette-wheel .layer-5, .ball-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: 380px 380px
}

.roulette-wheel .layer-2, .roulette-wheel .layer-4, .ball-container {
  will-change: transform
}

.roulette-wheel .layer-2 {
  background-image: url(../assets/roulette_2.png)
}

.roulette-wheel .layer-3 {
  background-image: url(../assets/roulette_3.png)
}

.roulette-wheel .layer-4 {
  background-image: url(../assets/roulette_4.png);
}

.roulette-wheel .layer-5 {
  background-image: url(../assets/roulette_5.png);
}

.roulette-wheel svg {
  position: absolute;
  top: 0;
  left: 0
}

.roulette-wheel circle {
  cursor: pointer;
  fill: transparent;
}

.roulette-wheel .ball {
  position: absolute;
  width: 14px;
  height: 14px;
  border-radius: 7px;
  background: #fff radial-gradient(circle at 5px 5px, #fff, #444);
  box-shadow: 1px 1px 4px #000;
  transform: translateY(-116px);
  top: 50%;
  left: 50%;
  margin: -7px;
  will-change: transform;
}

</style>
