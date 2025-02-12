<script setup>
import { ref, watch, onMounted, computed } from "vue";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import words from "./assets/words.json";

/**
 * State for the typing game
 * @typedef {Object} GameState
 */
const gameState = ref({
  targetString: null,
  targetArray: null,
  targetWordIndex: 0,
  correctStack: [],
  incorrectStack: [],
  text: "",
  isStarted: false,
  isFinished: false,
  wordCount: 25, // Default to 25 words
  startTime: null,
  endTime: null,
});

// Computed property to get the current target word
const targetWord = computed(() => {
  if (!gameState.value.targetArray) return "";
  return gameState.value.targetArray[gameState.value.targetWordIndex];
});

const coloredWords = computed(() => {
  if (!gameState.value.targetArray) return [];

  return gameState.value.targetArray.map((word, index) => {
    if (index < gameState.value.targetWordIndex || gameState.value.isFinished) {
      // Check if the word was typed correctly
      const wasCorrect = gameState.value.correctStack.includes(word);
      return {
        text: word,
        color: wasCorrect ? "text-green-600 font-medium" : "text-red-600 font-medium",
      };
    } else if (index === gameState.value.targetWordIndex) {
      return {
        text: word,
        color: "text-gray-500 font-medium",
      };
    } else {
      return {
        text: word,
        color: "text-black font-medium",
      };
    }
  });
});

const accuracy = computed(() => {
  if (!gameState.value.isFinished || !gameState.value.startTime || !gameState.value.endTime) {
    return "XX";
  }

  const total = gameState.value.correctStack.length + gameState.value.incorrectStack.length;
  if (total === 0) return 0;
  return Math.round((gameState.value.correctStack.length / total) * 100);
});

const wpm = computed(() => {
  if (!gameState.value.isFinished || !gameState.value.startTime || !gameState.value.endTime) {
    return "XX";
  }

  const timeInMinutes = (gameState.value.endTime - gameState.value.startTime) / 1000 / 60;
  const wordsTyped = gameState.value.correctStack.length + gameState.value.incorrectStack.length;
  return Math.round(wordsTyped / timeInMinutes);
});

const generateRandomTargetString = () => {
  const selectedWords = [];
  for (let i = 0; i < gameState.value.wordCount; i++) {
    const randomIndex = Math.floor(Math.random() * words.words.length);
    selectedWords.push(words.words[randomIndex]);
  }
  gameState.value.targetString = selectedWords.join(" ");
  gameState.value.targetArray = selectedWords;
};

const setWordCount = (count) => {
  gameState.value.wordCount = count;
  const gameInput = document.getElementById("gameInput");
  gameInput.focus();
  onRestart();
};

const getNextTargetWord = () => {
  gameState.value.targetWordIndex += 1;
  gameState.value.text = "";
};

const onFinished = () => {
  gameState.value.isFinished = true;
  const gameInput = document.getElementById("gameInput");
  gameInput.value = "";
  gameInput.disabled = true;
};

const onRestart = () => {
  gameState.value.isFinished = false;
  gameState.value.isStarted = false;
  gameState.value.startTime = null;
  gameState.value.endTime = null;
  gameState.value.text = "";
  gameState.value.correctStack = [];
  gameState.value.incorrectStack = [];
  gameState.value.targetWordIndex = 0;
  generateRandomTargetString();
};

// Whenever someone types in the input element
watch(
  () => gameState.value.text,
  (newText) => {
    // Start timer on first keystroke
    if (!gameState.value.isStarted && newText.length === 1) {
      gameState.value.isStarted = true;
      gameState.value.startTime = Date.now();
    }

    // For the last word, check immediately without requiring space
    if (
      gameState.value.targetWordIndex === gameState.value.targetArray.length - 1 &&
      newText === targetWord.value
    ) {
      gameState.value.correctStack.push(targetWord.value);
      gameState.value.endTime = Date.now();
      onFinished();
    }
    // For all other words, only check when space is pressed
    else if (newText.endsWith(" ")) {
      // Remove the trailing space and compare
      const enteredWord = newText.trim();
      if (enteredWord === targetWord.value) {
        gameState.value.correctStack.push(targetWord.value);
        getNextTargetWord();
      } else {
        gameState.value.incorrectStack.push(targetWord.value);
        getNextTargetWord();
      }
      gameState.value.text = "";
    }
  }
);

onMounted(() => {
  const gameInput = document.getElementById("gameInput");
  gameInput.focus();

  generateRandomTargetString();

  // Add event listener to input element that allows us to pop the last incorrect word when the incorrect stack > 0
  gameInput.addEventListener("keydown", (e) => {
    if (
      e.key === "Backspace" &&
      gameState.value.incorrectStack.length > 0 &&
      gameState.value.text === ""
    ) {
      const lastIncorrect = gameState.value.incorrectStack.pop();
      gameState.value.targetWordIndex = lastIncorrect.index;
      gameState.value.text = lastIncorrect.attemptedWord;
    }
  });
});
</script>

<template>
  <main id="game" class="flex flex-col justify-center max-w-lg min-h-screen mx-auto">
    <div class="flex flex-col gap-2">
      <section id="toolbar">
        <div class="flex items-center justify-between px-2">
          <div id="words-group" class="flex gap-2">
            <button
              id="word-option-10"
              :class="gameState.wordCount === 10 ? 'text-black font-bold' : 'text-gray-500'"
              @click="setWordCount(10)"
            >
              10
            </button>
            <span class="text-gray-500">/</span>
            <button
              id="word-option-25"
              :class="gameState.wordCount === 25 ? 'text-black font-bold' : 'text-gray-500'"
              @click="setWordCount(25)"
            >
              25
            </button>
            <span class="text-gray-500">/</span>
            <button
              id="word-option-50"
              :class="gameState.wordCount === 50 ? 'text-black font-bold' : 'text-gray-500'"
              @click="setWordCount(50)"
            >
              50
            </button>
            <span class="text-gray-500">/</span>
            <button
              id="word-option-100"
              :class="gameState.wordCount === 100 ? 'text-black font-bold' : 'text-gray-500'"
              @click="setWordCount(100)"
            >
              100
            </button>
          </div>
          <div id="stats" class="flex gap-2 text-gray-500">
            <p>WPM: {{ wpm }}</p>
            <span class="text-gray-500">/</span>
            <p>ACC: {{ accuracy }}</p>
          </div>
        </div>
      </section>
      <section id="game">
        <Card class="flex flex-col gap-4 p-4">
          <p class="flex flex-wrap gap-2 text-lg">
            <span v-for="(word, index) in coloredWords" :key="index" :class="word.color">
              {{ word.text }}
            </span>
          </p>
          <div class="flex items-center gap-2">
            <Input
              id="gameInput"
              type="text"
              v-model="gameState.text"
              :disabled="gameState.isFinished"
            />
            <Button @click="onRestart">Restart</Button>
          </div>
        </Card>
      </section>
    </div>
  </main>
</template>
