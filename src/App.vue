<script setup>
import words from "./words.js";
import { ref, computed } from "vue";

const dividend = ref(0);
const divisor = ref(1.09);
const guess = ref(0);
const results = ref([]);

const homeCurrency = ref("EUR");
const foreignCurrency = ref("USD");

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

const width = 800;
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
  <main class="">
    <div class="card m-4 bg-gray-900">
      <div class="card-body">
        <h2 class="card-title">Which conversion do you want to practice?</h2>
        <!-- name of first, name of second, conversion rate (free input fields) -->
        <input
          type="text"
          class="input p2 text-4xl"
          placeholder="Home currency"
          v-model="homeCurrency"
        />
        <input
          type="text"
          class="input p2 text-4xl"
          placeholder="Foreign currency"
          v-model="foreignCurrency"
        />
        <input
          type="number"
          class="input p2 text-4xl"
          placeholder="Conversion rate"
          v-model="divisor"
        />
      </div>
    </div>

    <div class="card m-4 bg-gray-900">
      <div class="card-body">
        <h2 class="card-title my-2 text-4xl">
          What is {{ dividend }} {{ foreignCurrency }} in {{ homeCurrency }}?
        </h2>
        <div :class="{ hidden: !isRevealed }" class="flex items-center gap-2">
          <p class="my-2 text-4xl">
            You guessed {{ guess }} {{ homeCurrency }}.
            <!-- round to two digits after point -->
            It's {{ (dividend / divisor).toFixed(2) }} {{ homeCurrency }}
          </p>
        </div>
        <div class="card-actions mt-6 pt-2 w-full">
          <input
            type="number"
            class="input p2 text-4xl"
            v-model="guess"
            v-if="!isRevealed"
          />
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

    <div class="card m-4 bg-gray-900">
      <div class="card-body">
        <h2 class="card-title">Your Guesses</h2>
        <p>Perfect guesses are on the green line.</p>

        <div class="max-w-screen-md" id="svg-wrapper">
          <svg :width="width" :height="height" class="p4">
            <!-- draw a horizontal green line at 0 -->
            <line
              :x1="margin"
              :y1="scaleY(0)"
              :x2="width - margin"
              :y2="scaleY(0)"
              stroke="green"
              stroke-width="5"
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
      </div>
    </div>
  </main>
</template>
