<template>
  <div id="app">
    <h1>ゆっくりポモドーロタイマー</h1>
    <template v-if="pomodoro">
      <p>{{ timeString(timer) }}</p>
      <button @click="countStart" v-if="interval === 0">Start</button>
      <button v-else-if="timer < 25 * minToSec" @click="stop">Stop</button>
      <button v-if="timer >= 25 * minToSec" @click="takeABreak">take a break!</button>
    </template>
    <template v-else>
      <p>{{ timeString(breakTimer) }}</p>
      <button @click="stop" v-if="breakTimer < 5 * minToSec">Stop</button>
      <button @click="countStart" v-else>Time to work!</button>
    </template>
  </div>
</template>

<script>
// ファイル名
const NAMES = {
  start: "01_start",
  min5: "02_05min",
  min10: "03_10min",
  min15: "04_15min",
  min20: "05_20min",
  min25: "06_25min",
  takeABreak: "07_take_a_break",
  next: "08_lets_start"
};

// 音声再生構造体
const Audios = {
  instance: null,
  constructor() {
    this.instance = new Audio();
  },
  setPath(str) {
    this.instance.src = `/assets/${str}.mp3`;
    return this;
  },
  play() {
    if (this.instance !== undefined) {
      this.instance.play();
    }
  }
};

export default {
  data() {
    return {
      pomodoro: true,
      timer: 0,
      interval: 0,
      breakTimer: 0,
      breakInterval: 0,
      audio: null,
      minToSec: 60 // デバッグ中は1にすると楽
    };
  },
  mounted() {
    this.audio = Audios.constructor();
  },
  methods: {
    countStart() {
      this.pomodoro = true;
      clearInterval(this.breakInterval);
      this.breakTimer = 0;

      this.interval = setInterval(() => {
        this.timer++;
      }, 1000);
      Audios.setPath(NAMES.start).play();
    },
    stop() {
      clearInterval(this.interval);
      this.interval = 0;
      clearInterval(this.breakInterval);
      this.breakInterval = 0;

      this.timer = 0;
      this.breakTimer;
      this.pomodoro = true;
    },
    takeABreak() {
      this.pomodoro = false;
      clearInterval(this.interval);
      this.timer = 0;
      this.breakInterval = setInterval(() => {
        this.breakTimer++;
      }, 1000);
      Audios.setPath(NAMES.takeABreak).play();
    },
    timeString(time) {
      return `${("0" + Math.floor(time / 60)).slice(-2)}:${("0" + time).slice(
        -2
      )}`;
    }
  },
  watch: {
    // カウント中に音を鳴らす
    timer(t) {
      switch (t) {
        case 5 * this.minToSec:
          Audios.setPath(NAMES.min5).play();
          break;
        case 10 * this.minToSec:
          Audios.setPath(NAMES.min10).play();
          break;
        case 15 * this.minToSec:
          Audios.setPath(NAMES.min15).play();
          break;
        case 20 * this.minToSec:
          Audios.setPath(NAMES.min20).play();
          break;
        case 25 * this.minToSec:
          Audios.setPath(NAMES.min25).play();
          break;
      }
    },
    breakTimer(t) {
      switch (t) {
        case 5 * this.minToSec:
          Audios.setPath(NAMES.next).play();
          break;
      }
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
