<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from "vue";

const dividend = ref(0);
const divisor = ref(1.2);
const guess = ref(0);
const results = ref([]);

const homeCurrency = ref("EUR");
const foreignCurrency = ref("USD");

let isRevealed = ref(false);
const guessInput = ref(null);

function handleGlobalKeydown(e) {
  if (e.key === "Enter" && isRevealed.value) {
    evaluateScore();
  }
}

onMounted(() => {
  window.addEventListener("keydown", handleGlobalKeydown);
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleGlobalKeydown);
});

generateRandomExercise();

function generateRandomExercise() {
  // set dividend to random between 800 and 5
  dividend.value = Math.floor(Math.random() * (800 - 5 + 1) + 5);
  guess.value = null;
  isRevealed.value = false;
  nextTick(() => guessInput.value?.focus());
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

const width = 400;
const height = 200;
const margin = 24;

const currentError = computed(() => {
  if (guess.value == null || !isRevealed.value) return null;
  const correct = dividend.value / divisor.value;
  return ((guess.value - correct) / correct) * 100;
});

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
  <main class="min-h-screen bg-base-300 p-4 md:p-8">
    <div class="mx-auto max-w-lg flex flex-col gap-4">

      <!-- Config: collapsible, compact -->
      <details class="collapse collapse-arrow bg-base-200 rounded-box">
        <summary class="collapse-title text-sm font-medium py-3 min-h-0">
          {{ foreignCurrency }} → {{ homeCurrency }} @ {{ divisor }}
        </summary>
        <div class="collapse-content flex flex-col gap-2 pb-4">
          <div class="flex gap-2">
            <input
              type="text"
              class="input input-sm flex-1"
              placeholder="Foreign"
              v-model="foreignCurrency"
            />
            <input
              type="text"
              class="input input-sm flex-1"
              placeholder="Home"
              v-model="homeCurrency"
            />
          </div>
          <input
            type="number"
            class="input input-sm"
            placeholder="Rate"
            v-model="divisor"
            step="0.01"
          />
        </div>
      </details>

      <!-- Exercise: prominent -->
      <div class="card bg-base-200">
        <div class="card-body gap-4 p-6">
          <h2 class="text-2xl md:text-3xl font-bold">
            {{ dividend }} {{ foreignCurrency }} = ?
          </h2>

          <div v-if="isRevealed" class="text-lg opacity-80">
            You guessed <span class="font-semibold">{{ guess }}</span>.
            It's <span class="font-semibold text-primary">{{ (dividend / divisor).toFixed(2) }}</span> {{ homeCurrency }}.
            <span class="block text-sm mt-1" :class="Math.abs(currentError) < 1 ? 'text-success' : ''">
              <template v-if="Math.abs(currentError) < 1">Spot on!</template>
              <template v-else>{{ Math.abs(currentError).toFixed(0) }}% too {{ currentError > 0 ? 'high' : 'low' }}</template>
            </span>
          </div>

          <div class="flex gap-2 mt-2">
            <input
              ref="guessInput"
              type="number"
              class="input input-bordered flex-1 text-xl"
              :placeholder="homeCurrency"
              v-model="guess"
              v-if="!isRevealed"
              @keyup.enter="guess != null && (isRevealed = true)"
            />
            <button
              class="btn btn-primary"
              @click="isRevealed = true"
              v-if="!isRevealed"
            >
              Check <kbd class="kbd kbd-xs ml-1">↵</kbd>
            </button>
            <button class="btn btn-primary flex-1" @click="evaluateScore" v-else>
              Next <kbd class="kbd kbd-xs ml-1">↵</kbd>
            </button>
          </div>
        </div>
      </div>

      <!-- Graph: minimal -->
      <div class="card bg-base-200" v-if="results.length || isRevealed">
        <div class="card-body p-4">
          <svg
            :viewBox="`0 0 ${width} ${height}`"
            class="w-full h-auto"
            preserveAspectRatio="xMidYMid meet"
          >
            <!-- Zero line -->
            <line
              :x1="margin"
              :y1="scaleY(0)"
              :x2="width - margin"
              :y2="scaleY(0)"
              class="stroke-base-content/20"
              stroke-width="1"
            />

            <!-- Data points -->
            <circle
              v-for="(point, index) in results.slice(-20)"
              :key="index"
              :cx="scaleX(index)"
              :cy="scaleY(point.missedByPercent)"
              r="4"
              class="fill-primary"
            />

            <!-- Current guess (white) -->
            <circle
              v-if="isRevealed && currentError != null"
              :cx="scaleX(results.length)"
              :cy="scaleY(currentError)"
              r="5"
              fill="white"
            />
          </svg>
        </div>
      </div>

    </div>
  </main>
</template>
