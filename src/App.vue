<script setup>
import words from "./words.js";
import { ref, computed } from "vue";

const dividend = ref(0);
const divisor = ref(33.49);
const guess = ref(0);
const results = ref([]);
let unitsPracticedToday = 0;
let unitsPracticedYesterday = 0;

// same with localStorage stats
let stats = {};
if (!localStorage.getItem("stats")) {
  stats = {
    counter: 0,
  };
} else {
  stats = JSON.parse(localStorage.getItem("stats"));
}

let isRevealed = ref(false);

let valueEase = ref(null);
let valueCorrect = ref(null);
let valueAnki = ref(null);

generateRandomExercise();

function generateRandomExercise() {
  // set dividend to random between 800 and 5
  dividend.value = Math.floor(Math.random() * (800 - 5 + 1) + 5);
  guess.value = null;
  isRevealed.value = false;
}

function evaluateScore() {
  console.log("guess", guess.value);
  results.value.push({
    date: new Date().toISOString(),
    dividend: dividend.value,
    divisor: divisor.value,
    guess: guess.value,
    correct: dividend.value / divisor.value,
    // missedByPercent: how much the guess was over or under the correct result, expressed as percent of the correct result
    missedByPercent:
      ((guess.value - dividend.value / divisor.value) /
        (dividend.value / divisor.value)) *
      100,
  });
  generateRandomExercise();
}

// function evaluateScore() {
//   if (randomWord.value.evaluationType == "anki") {
//     if (valueAnki.value == null) return;
//     // add a log to the word's repetition property - if the property doesn't exist, create it
//     if (!randomWord.value.repetitions) {
//       randomWord.value.repetitions = [];
//     }
//     randomWord.value.repetitions.push({
//       date: new Date().toISOString(),
//       score: valueAnki.value,
//     });
//     stats.counter++;
//     localStorage.setItem("stats", JSON.stringify(stats));
//     getRandomWord();
//   } else {
//     if (valueEase.value == null || valueCorrect.value == null) return;
//     // add a log to the word's repetition property - if the property doesn't exist, create it
//     if (!randomWord.value.repetitions) {
//       randomWord.value.repetitions = [];
//     }
//     randomWord.value.repetitions.push({
//       date: new Date().toISOString(),
//       ease: valueEase.value,
//       correct: valueCorrect.value,
//     });
//     stats.counter++;
//     localStorage.setItem("stats", JSON.stringify(stats));
//     getRandomWord();
//   }
// }

const width = 800
const height = 400;
const margin = 40;

// use the numbers of objects in results as maxX, but maximally the last 20
const maxX = computed(() => Math.max(20, results.value.length));
const maxY = computed(() => {
  // get the highest missedByPercent value from results
  const max = Math.max(...results.value.map((point) => point.missedByPercent));
  // round up to the next 10
  return Math.ceil(max / 10) * 10;
});

const minY = computed(() => {
  // get the lowest missedByPercent value from results
  const min = Math.min(...results.value.map((point) => point.missedByPercent));
  // round down to the next 10
  return Math.floor(min / 10) * 10;
});

const scaleX = computed(() => {
  const xRange = maxX.value;
  return (x) => (x / xRange) * (width - 2 * margin) + margin;
});

// this doesn't need to be computed, but I don't actually understand the function :)
const scaleY = computed(() => {
  const yRange = maxY.value - minY.value;
  return (y) =>
    height - ((y - minY.value) / yRange) * (height - 2 * margin) - margin;
});
</script>

<template>
  <!-- <small> Practiced {{ stats.counter }} times so far </small> -->
  <div
    class="card bg-gray-600 shadow-xl m-4 flex flex-col items-center w-full max-w-screen-xl"
  >
    <div class="card-body">
      <h2 class="card-title my-2 text-6xl">{{ dividend }} EGP in EUR?</h2>
      <div :class="{ hidden: !isRevealed }" class="flex items-center gap-2">
        <p class="my-2 text-4xl">
          <!-- round to two digits after point -->
          It's {{ (dividend / divisor).toFixed(2) }} EUR
        </p>
      </div>
      <div class="card-actions justify-end mt-6 pt-2">
        <input type="number" class="input p2 text-4xl" v-model="guess" />
        <button
          class="btn btn-primary"
          @click="isRevealed = true"
          v-if="!isRevealed"
        >
          Check
        </button>
        <button class="btn btn-primary" @click="evaluateScore" v-else>
          Next Exercise
        </button>
      </div>
    </div>
  </div>

  <!-- {{ results }} -->

  <div class="max-w-screen-md" id="svg-wrapper">
    <svg :width="width" :height="height" class="p4">
      <!-- Draw x-axis -->
      <line
        :x1="margin"
        :y1="height - margin"
        :x2="width - margin"
        :y2="height - margin"
        stroke="black"
      />

      <!-- draw a horizontal green line at 0 -->
      <line
        :x1="margin"
        :y1="scaleY(0)"
        :x2="width - margin"
        :y2="scaleY(0)"
        stroke="green"
      />

      <!-- Draw y-axis -->
      <line
        :x1="margin"
        :y1="margin"
        :x2="margin"
        :y2="height - margin"
        stroke="black"
      />

      <!-- Draw data points -->
      <!-- maximally use the last 20 results -->
      <!-- get x value from position in results -->
      <!-- get y value from missedByPercent -->
      <circle
        v-for="(point, index) in results.slice(-20)"
        :key="index"
        :cx="scaleX(index)"
        :cy="scaleY(point.missedByPercent)"
        :r="12"
        fill="#7a29e9"
      />
    </svg>
  </div>
</template>

<style scoped></style>
