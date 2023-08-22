<script setup>
import words from "./words.js";
import { ref } from "vue";

let dividend = ref(0);
let divisor = ref(29);
let guess = ref(0);
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
        <button class="btn btn-primary" @click="generateRandomExercise" v-else>
          Next Exercise
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
