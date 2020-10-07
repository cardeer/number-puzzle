<template>
  <div class="puzzle">
    <div class="puzzle-render">
      <div class="lines">
        <span
          v-for="i in state.puzzleSize - 1"
          :key="i"
          class="line v-line"
          :style="{ left: `${(i * 100) / state.puzzleSize}%` }"
        ></span>
        <span
          v-for="i in state.puzzleSize - 1"
          :key="i"
          class="line h-line"
          :style="{ top: `${(i * 100) / state.puzzleSize}%` }"
        ></span>
      </div>

      <div class="numbers">
        <div v-for="(set, i) in state.map" :key="i">
          <div
            v-for="(number, j) in set"
            :key="'num' + number"
            class="number"
            :class="number == 0 && 'hide'"
            :style="{
              width: `calc(100% / ${state.puzzleSize})`,
              height: `calc(100% / ${state.puzzleSize})`,
              top: `${i * (100 / state.puzzleSize)}%`,
              left: `${j * (100 / state.puzzleSize)}%`,
            }"
          >
            <span class="number-preview">{{ number }}</span>
          </div>
        </div>
      </div>
    </div>

    <p>{{ startedTime }}</p>

    <div class="controls">
      <button
        v-if="!state.started"
        style="background-color: #38b73c; color: white"
        @click="start"
      >
        Start
      </button>
      <button
        v-if="state.started"
        style="background-color: #b73838; color: white"
        @click="stop"
      >
        Stop
      </button>
    </div>
  </div>

  <div class="options">
    <h3>Game settings</h3>
    <p>Size: {{ state.puzzleSize + " x " + state.puzzleSize }}</p>
    <input v-model="state.puzzleSize" type="range" min="3" max="10" />
  </div>
</template>

<script>
import { reactive, onMounted, onBeforeUnmount, watch, computed } from "vue";
import "@/assets/css/puzzle.css";

export default {
  name: "Home",
  setup() {
    const state = reactive({
      puzzleSize: 4,
      started: false,
      time: 0,
      map: [],
      validPosition: {},
    });

    const square = computed(() => state.puzzleSize * state.puzzleSize);
    const startedTime = computed(
      () =>
        `${("0" + Math.floor((state.time % 3600) / 60)).substr(-2)}:${(
          "0" + Math.floor(state.time % 60)
        ).substr(-2)}`
    );

    const moveDirections = {
      37: { x: 1, y: 0 },
      38: { x: 0, y: 1 },
      39: { x: -1, y: 0 },
      40: { x: 0, y: -1 },
    };
    var empty = {
      y: state.puzzleSize - 1,
      x: state.puzzleSize - 1,
    };
    var interval = 0;
    var validCount = 0;

    function createMap() {
      const numbers = [...Array(square.value).keys()];
      const data = [];

      for (let i = 0; i < state.puzzleSize; i++) {
        const set = [];
        for (let j = 0; j < state.puzzleSize; j++) {
          const randomIndex = Math.floor(Math.random() * numbers.length);
          set.push(numbers[randomIndex]);

          if (numbers[randomIndex] == 0) {
            empty = {
              y: i,
              x: j,
            };
          }

          numbers.splice(randomIndex, 1);
        }
        data.push(set);
      }

      state.map = data;
    }

    function start() {
      state.validPosition = {};
      validCount = 0;
      state.time = 0;

      createMap();

      state.started = true;
      interval = setInterval(() => state.time++, 1000);
    }

    function stop() {
      state.started = false;
      clearInterval(interval);
    }

    function move(e) {
      if (moveDirections[e.keyCode] && state.started) {
        e.preventDefault();
        const direction = moveDirections[e.keyCode];
        const fromY = empty.y + direction.y;
        const fromX = empty.x + direction.x;
        const toY = empty.y;
        const toX = empty.x;

        if (state.map[fromY] && state.map[fromY][fromX]) {
          const toNumber = toY * state.puzzleSize + toX + 1;
          const fromNumber = state.map[fromY][fromX];

          state.map[toY][toX] = state.map[fromY][fromX];
          state.map[fromY][fromX] = 0;
          empty.y += direction.y;
          empty.x += direction.x;

          if (fromNumber == toNumber) {
            state.validPosition[toNumber] = true;
            validCount++;
          } else {
            if (state.validPosition[fromNumber]) {
              delete state.validPosition[fromNumber];
              validCount--;
            }
          }

          if (validCount == square.value - 1) {
            stop();
          }
        }
      }

      if (e.keyCode == 13) {
        stop();
      }
    }

    onMounted(() => {
      window.addEventListener("keydown", move);
    });

    onBeforeUnmount(() => {
      window.removeEventListener("keydown", move);
      clearInterval(interval);
    });

    watch(
      () => state.puzzleSize,
      (b, a) => {
        if (b != a) {
          state.started = false;
          state.map = [];
        }
      }
    );

    return { state, startedTime, start, stop };
  },
};
</script>
