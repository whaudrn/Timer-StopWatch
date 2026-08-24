<script setup>

import './index.css'
import './reset.css'
import { ref } from 'vue';

const activeTab = ref('timer')

const timerMin = ref('00')
const timerSec = ref('00')
const totalSec = ref(0);
const isRunning = ref(false)
const isPause = ref(false)
const progressRef = ref(null)

const stopWatchMin = ref('00')
const stopWatchSec = ref('00')
const stopWatchMiliSec = ref('00')
const stopWatchRunning = ref(false)
const count = ref(0);
const rest = ref(0);
const markerRef = ref(null)
const isLabs = ref(false)
const currentTime = ref(null)
const labList = ref([])

let progressAni;
let timerSet;

const computeTime = (sec) => {
  totalSec.value += sec
  if (progressAni) {
    progressAni.cancel();
    progressAni = null;
  }
  displayTime()
  if (isRunning.value) {
    progress(totalSec.value)
  }
}

const displayTime = () => {
  timerMin.value = String(Math.floor(totalSec.value / 60)).padStart(2, "0");
  timerSec.value = String(totalSec.value % 60).padStart(2, "0");
}

const toggleTimer = () => {
  if (totalSec.value !== 0) {
    isPause.value = !isPause.value
    controlAnimation()
  } else {
    alert("시간 입력")
  }
}

const setTime = () => {
  const tick = () => {
    if (totalSec.value !== 0) {
      totalSec.value--;
      displayTime(totalSec.value);
    } else {
      clearInterval(timerSet);
      progressAni = null;
      console.log("타이머종료!");
      isRunning.value = false;
      isPause.value = false;
    }
  }
  tick();
  timerSet = setInterval(tick, 1000)
}

const controlAnimation = () => {
  console.log(totalSec.value);
  if (!isRunning.value) {
    isRunning.value = true;

    setTime()

    if (!progressAni) {
      console.log("1");
      progress(totalSec.value);
    } else {
      console.log("2");
      progressAni.play();
    }

  } else {
    isRunning.value = false;
    progressAni.pause();
    clearInterval(timerSet);
  }
}

const progress = () => {
  progressAni = progressRef.value.animate(
    [
      { strokeDashoffset: "1256", stroke: "#0B57CF" },
      { strokeDashoffset: "0", stroke: "#0B57CF" }
    ],
    {
      duration: totalSec.value * 1000,
      easing: "linear",
      iterations: 1,
    }
  );
}

const reset = () => {
  timerMin.value = '00'
  timerSec.value = '00'
  clearInterval(timerSet);
  totalSec.value = 0;
  isPause.value = false;
  isRunning.value = false;
  if (progressAni) {
    progressAni.cancel();
    progressAni = null;
  }
}

const stopWatchTimeCompute = () => {
  let restStr;
  rest.value = count.value / 6000;
  stopWatchMin.value = String(Math.floor(rest.value)).padStart(2, '0');
  restStr = String(count.value % 6000).padStart(4, '0')
  stopWatchSec.value = restStr.slice(0, 2)
  stopWatchMiliSec.value = restStr.slice(2, 4)
}
let stopWatchSetTime;
const toggleStopWatch = () => {
  if (!stopWatchRunning.value) {
    stopWatchRunning.value = true;
    if (markerAni) {
      markerAni.play()
    } else {
      markerAnimation()
    }
    stopWatchSetTime = setInterval(() => {
      stopWatchTimeCompute()
      count.value++;
    }, 10)
  } else {
    markerAni.pause()
    stopWatchRunning.value = false;
    clearInterval(stopWatchSetTime)
  }
}
const stopWatchReset = () => {
  stopWatchRunning.value = false;
  stopWatchMin.value = '00'
  stopWatchSec.value = '00'
  stopWatchMiliSec.value = '00'
  count.value = 0
  clearInterval(stopWatchSetTime)
  if (markerAni) {
    markerAni.cancel()
    markerAni = null
  }
  isLabs.value = false;
  labList.value = [];
}
let markerAni;
const markerAnimation = () => {
  markerAni = markerRef.value.animate([
    { transform: 'rotate(-90deg) translate(40%)' },
    { transform: 'rotate(270deg) translate(40%)' }
  ],
    {
      easing: 'linear',
      duration: 4000,
      iterations: Infinity,
      fill: 'forwards'
    }
  )
}

const clickLabs = () => {
  if (!isLabs.value && stopWatchRunning.value) {
    if (markerAni) {
      markerAni.cancel();
      markerAni = null;
    }
    markerAnimation();
  }
  isLabs.value = true;
  labList.value.push(currentTime.value.textContent)
  console.log("기록");
  console.log(currentTime.value.textContent);
  console.log(labList.value);
}
const delLabs = () => {
  labList.value = []
}
</script>

<template>
  <div class="wrap">

    <header>
      <button :class="{ on: activeTab === 'timer' }" @click="activeTab = 'timer'">Timer</button>
      <button :class="{ on: activeTab === 'stopWatch' }" @click="activeTab = 'stopWatch'">stopWatch</button>
    </header>

    <main>

      <section class="timer" v-show="activeTab === 'timer'">
        <div class="circle">
          <svg width="500" height="500" class="svg">
            <circle r="200" cx="50%" cy="50%" class="border"></circle>
            <circle r="200" cx="50%" cy="50%" class="progress" ref="progressRef"></circle>
          </svg>
          <div class="inner">
            <p><span class="timerMin">{{ timerMin }}</span>:<span class="timerSec">{{ timerSec }}</span></p>
            <div class="addTimeBtn">
              <button class="addTen" @click="computeTime(3)">+ 3 Sec</button>
              <button class="addThirty" @click="computeTime(10)">+ 10 Sec</button>
            </div>
          </div>
        </div>
        <div class="buttonWrap">
          <div class="startPause">
            <button @click="toggleTimer">{{ isPause ? "Pause" : "Start" }}</button>
          </div>
          <div class="reset">
            <button @click="reset">초기화</button>
          </div>
        </div>
      </section>

      <section class="stopwatch" v-show="activeTab === 'stopWatch'">
        <div class="display">
          <div class="circle">
            <p ref="currentTime">
              <span class="min">{{ stopWatchMin }}</span>:<span class="sec">{{ stopWatchSec }}</span>:<span
                class="miliSec">{{ stopWatchMiliSec }}</span>
            </p>
            <svg :width="isLabs ? 300 : 500" :height="isLabs ? 300 : 500" class="svg">
              <circle :r="isLabs ? 120 : 200" cx="50%" cy="50%" class="border"></circle>
              <circle r="10" cx="50%" cy="50%" class="marker" ref="markerRef"></circle>
            </svg>
          </div>
        </div>
        <div class="buttonWrap">
          <div class="startPause">
            <button @click="toggleStopWatch">{{ stopWatchRunning ? 'Pause' : 'Start' }}</button>
          </div>
          <div class="reset">
            <button @click="stopWatchReset">초기화</button>
          </div>
          <div class="labs">
            <button @click="clickLabs">랩 기록</button>
            <button @click="delLabs">기록 삭제</button>
          </div>
        </div>
        <div v-show="isLabs" class="labList">
          <ul>
            <li v-for="(list, idx) in labList.slice(0, 50)" key="idx">{{ idx }} : {{ list }}</li>
          </ul>
        </div>
      </section>

    </main>
  </div>
</template>

<style scoped>
.border {
  fill: #fff;
}
</style>
