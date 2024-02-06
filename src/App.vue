<template>
  <div
    class="w-dvh flex h-dvh items-center justify-center bg-amber-50 text-amber-950"
  >
    <div
      class="relative flex h-full max-h-[900px] w-full max-w-[400px] flex-col items-center justify-between gap-16 overflow-clip bg-amber-100 px-16 py-12 text-center lg:rounded-2xl"
    >
      <div
        class="absolute top-0 flex h-12 w-full items-center justify-evenly bg-amber-200 px-4"
      >
        <div
          v-for="lang in languageOptions"
          class="cursor-pointer rounded-md px-4 py-1 transition-all"
          @click="switchLanguage(lang)"
          :class="
            selectedLang === lang
              ? 'bg-white hover:bg-white'
              : 'bg-white/30 hover:bg-white/80'
          "
        >
          {{ lang.englishName }}
        </div>
      </div>

      <h1 class="mt-12 text-3xl">
        {{ selectedLang?.titleText }}
      </h1>
      <textarea
        v-model="message"
        :placeholder="selectedLang?.placeholderText"
        :class="inputFontSize"
        class="my-auto h-full w-full resize-none rounded border-none bg-transparent p-5 text-center font-light placeholder-inherit outline-none"
      ></textarea>
      <div
        @mousedown="startRecognition"
        @touchstart="startRecognition"
        @mouseup="stopRecognition"
        @touchend="stopRecognition"
        class="group flex h-24 w-24 shrink-0 items-center justify-center rounded-full bg-gradient-to-r from-slate-50 to-slate-300 transition-all hover:cursor-pointer"
        :class="
          isListening === true
            ? 'shadow-[0_3px_5px_0_rgba(0,0,0,0.3)]'
            : 'shadow-[0_8px_10px_0_rgba(38,38,38,0.3)]'
        "
      >
        <MicrophoneIcon
          class="h-12 w-12 transition-all group-hover:fill-amber-500/10"
          :class="isListening ? '!fill-amber-900' : 'fill-gray-800/10'"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import { MicrophoneIcon } from "@heroicons/vue/24/solid";

interface LanguageOption {
  code: string;
  localName: string;
  englishName: string;
  titleText: string;
  placeholderText: string;
  canSpeak: boolean;
}

const languageOptions = ref<LanguageOption[]>([
  {
    code: "en-US",
    localName: "English",
    englishName: "English",
    titleText: "Dictate and listen to what you just said.",
    placeholderText: "Press down the button whilst speaking",
    canSpeak: true,
  },
  {
    code: "de-DE",
    localName: "Deutsch",
    englishName: "German",
    titleText: "Diktieren und anhören, was Sie gerade gesagt haben.",
    placeholderText: "Drücken Sie die Taste, während Sie sprechen.",
    canSpeak: true,
  },
  {
    code: "it-IT",
    localName: "italiano",
    englishName: "Italian",
    titleText: "Parla e ascolta ciò che hai appena detto.",
    placeholderText: "Premi il pulsante mentre parli.",
    canSpeak: true,
  },
  {
    code: "da-DK",
    localName: "dansk",
    englishName: "Danish",
    titleText: "Dikter og lyt til hvad du lige har sagt.",
    placeholderText: "Tryk på knappen mens du taler.",
    canSpeak: false,
  },
]);

const message = ref<string>("");
const selectedLang = ref<LanguageOption>(languageOptions.value[0]);
const isListening = ref<boolean>(false);
let recognition: any;

const inputFontSize = computed(() => {
  return {
    "text-2xl": message.value.length > 0 && message.value.length <= 25,
    "text-lg": message.value.length > 25 && message.value.length <= 35,
    "text-base": message.value.length > 35 && message.value.length <= 40,
    "text-sm": message.value.length > 40,
  };
});

function initializeSpeechRecognition() {
  if ("webkitSpeechRecognition" in window || "SpeechRecognition" in window) {
    //@ts-ignore
    recognition = new (webkitSpeechRecognition || SpeechRecognition)();
    recognition.interimResults = true;
    recognition.continuous = true;
    recognition.lang = selectedLang.value!.code;
  } else {
    console.error("Speech recognition is not supported in this browser.");
  }
}

function startRecognition() {
  recognition.start();
  isListening.value = true;

  recognition.onresult = (event: any) => {
    message.value = event.results[0][0].transcript;
  };

  recognition.onend = () => {
    recognition.stop();
    isListening.value = false;
  };
}

function stopRecognition() {
  recognition.stop();
  isListening.value = false;
  if (selectedLang.value!.canSpeak === false) {
    console.log("This language cannot be synthethised. Try another language.");
  } else {
    readOutMessage();
  }
}

function readOutMessage() {
  const utterance = new SpeechSynthesisUtterance();
  utterance.text = message.value;
  utterance.lang = selectedLang.value!.code;
  speechSynthesis.speak(utterance);
}

function switchLanguage(lang: LanguageOption) {
  selectedLang.value = lang;
  message.value = "";
  initializeSpeechRecognition();
}

onMounted(() => {
  initializeSpeechRecognition();
});
</script>

<style scoped></style>
