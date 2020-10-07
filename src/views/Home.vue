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
    <input v-model="state.puzzleSize" type="range" min="2" max="10" />
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
      map: [],
    });

    const square = computed(() => state.puzzleSize * state.puzzleSize);

    var empty = {
      y: state.puzzleSize - 1,
      x: state.puzzleSize - 1,
    };
    const moveDirections = {
      37: { x: 1, y: 0 },
      38: { x: 0, y: 1 },
      39: { x: -1, y: 0 },
      40: { x: 0, y: -1 },
    };

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
      createMap();
      state.started = true;
    }

    function stop() {
      state.started = false;
    }

    function move(e) {
      if (moveDirections[e.keyCode] && state.started) {
        e.preventDefault();
        const direction = moveDirections[e.keyCode];
        if (
          state.map[empty.y + direction.y] &&
          state.map[empty.y + direction.y][empty.x + direction.x]
        ) {
          state.map[empty.y][empty.x] =
            state.map[empty.y + direction.y][empty.x + direction.x];
          state.map[empty.y + direction.y][empty.x + direction.x] = 0;
          empty.y += direction.y;
          empty.x += direction.x;
        }
      }

      if (e.keyCode == 13) {
        state.started = false;
      }
    }

    onMounted(() => {
      window.addEventListener("keydown", move);
    });

    onBeforeUnmount(() => {
      window.removeEventListener("keydown", move);
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

    return { state, start, stop };
  },
};
</script>
