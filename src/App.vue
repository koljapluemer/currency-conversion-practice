<script setup>
import words from "./words.js";
import { ref, computed } from 'vue';

let dividend = ref(0);
let divisor = ref(29);
let guess = ref(0);
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
  isRevealed.value = false;
}

function evaluateScore() {
  results.value.push({
    date: new Date().toISOString(),
    dividend: dividend.value,
    divisor: divisor.value,
    guess: guess.value,
    correct: dividend.value / divisor.value,
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


const width = 400;
const height = 300;
const margin = 20;

const dataPoints = ref([
  { x: 10, y: 50 },
  { x: 30, y: 70 },
  { x: 50, y: 100 },
  { x: 70, y: 40 },
  { x: 90, y: 120 },
]);

const maxX = computed(() => Math.max(...dataPoints.value.map(point => point.x)));
const maxY = computed(() => Math.max(...dataPoints.value.map(point => point.y)));

const scaleX = computed(() => {
  const xRange = maxX.value;
  return x => (x / xRange) * (width - 2 * margin) + margin;
});

const scaleY = computed(() => {
  const yRange = maxY.value;
  return y => (y / yRange) * (height - 2 * margin) + margin;
});
</script>

<template>
  <!-- <small> Practiced {{ stats.counter }} times so far </small> -->
  <div
    class="card bg-gray-600 shadow-xl m-4 flex flex-col items-center w-full max-w-screen-xl"
  >
    <div class="card-body">
      <h2 class="card-title my-2 text-6xl">
        {{ dividend }} : {{ divisor }} = ?
      </h2>
      <div :class="{ hidden: !isRevealed }" class="flex items-center gap-2">
        <p class="my-2 text-4xl">
          {{ dividend }} : {{ divisor }} = {{ dividend / divisor }}
        </p>
      </div>
      <div class="card-actions justify-end mt-6 pt-2">
      <input type="number" class="input p2 text-4xl" v-bind="guess">
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

  {{ results }}


    <div>
    <svg :width="width" :height="height">
      <!-- Draw x-axis -->
      <line
        :x1="margin"
        :y1="height - margin"
        :x2="width - margin"
        :y2="height - margin"
        stroke="black"
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
      <circle
        v-for="(point, index) in dataPoints"
        :key="index"
        :cx="scaleX(point.x)"
        :cy="height - scaleY(point.y)"
        :r="4"
        fill="blue"
      />
    </svg>
  </div>
</template>

<style scoped></style>
