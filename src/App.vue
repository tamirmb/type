<script setup>
import { ref, watch, onMounted, computed } from "vue";
import Cookies from "js-cookie";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";
import words from "./assets/words.json";
import Toolbar from "@/components/Toolbar.vue";

const COOKIE_OPTIONS = {
  expires: 30,
  sameSite: "Lax",
};

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
  wordCount: Cookies.get("wordCount") || 25,
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
  Cookies.set("wordCount", count, COOKIE_OPTIONS);
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

    // Only process word completion when space is pressed or when the last word matches exactly
    const isLastWord = gameState.value.targetWordIndex === gameState.value.targetArray.length - 1;
    const enteredWord = newText.trim();

    // Handle last word matching exactly (no space needed)
    if (isLastWord && newText === targetWord.value) {
      gameState.value.correctStack.push(targetWord.value);
      gameState.value.endTime = Date.now();
      onFinished();
      return;
    }

    // Handle word completion on space
    if (newText.endsWith(" ") && enteredWord !== "") {
      if (enteredWord === targetWord.value) {
        gameState.value.correctStack.push(targetWord.value);
      } else {
        gameState.value.incorrectStack.push(targetWord.value);
      }

      // If this was the last word, end the game
      if (isLastWord) {
        gameState.value.endTime = Date.now();
        onFinished();
      } else {
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

  gameInput.addEventListener("keydown", (e) => {
    // Prevent space if there are no ASCII characters
    if (e.key === " " && (!gameState.value.text || !/[a-zA-Z0-9]/.test(gameState.value.text))) {
      e.preventDefault();
      return;
    }
  });
});
</script>

<template>
  <main id="game" class="flex flex-col justify-center max-w-lg min-h-screen mx-auto">
    <div class="flex flex-col gap-2">
      <Toolbar
        :wordCount="gameState.wordCount"
        :wpm="wpm"
        :accuracy="accuracy"
        @setWordCount="setWordCount"
      />
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
