<script setup>
defineProps({
  wordCount: {
    type: Number,
    required: true,
  },
  wpm: {
    type: [String, Number],
    required: true,
  },
  accuracy: {
    type: [String, Number],
    required: true,
  },
});

const emit = defineEmits(["setWordCount"]);

const onSelectWordCount = (count) => {
  emit("setWordCount", count);
};
</script>

<template>
  <section id="toolbar">
    <div class="flex items-center justify-between px-2">
      <div id="words-group" class="flex gap-2">
        <button
          id="word-option-10"
          :class="parseInt(wordCount) === 10 ? 'text-black font-bold' : 'text-gray-500'"
          @click="onSelectWordCount(10)"
        >
          10
        </button>
        <span class="text-gray-500">/</span>
        <button
          id="word-option-25"
          :class="parseInt(wordCount) === 25 ? 'text-black font-bold' : 'text-gray-500'"
          @click="onSelectWordCount(25)"
        >
          25
        </button>
        <span class="text-gray-500">/</span>
        <button
          id="word-option-50"
          :class="parseInt(wordCount) === 50 ? 'text-black font-bold' : 'text-gray-500'"
          @click="onSelectWordCount(50)"
        >
          50
        </button>
        <span class="text-gray-500">/</span>
        <button
          id="word-option-100"
          :class="parseInt(wordCount) === 100 ? 'text-black font-bold' : 'text-gray-500'"
          @click="onSelectWordCount(100)"
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
</template>
