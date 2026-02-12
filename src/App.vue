<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

//------------------------------------------------- State & Refs -------------------------------------------------//

// for speech recognition
const isListening = ref(false);
const interimTranscript = ref("");
const finalTranscript = ref("");
let recognition = null;

//tts
const text = ref("Hello, please introduce yourself.");
const voices = ref([]);
const selectedVoiceIndex = ref(0);
let synth = window.speechSynthesis;
// ---------------------------------------------- Methods ----------------------------------------------//
function startListening() {
  if (!recognition) return;
  interimTranscript.value = "";
  finalTranscript.value = "";
  recognition.start();
}

function stopListening() {
  if (!recognition) return;
  recognition.stop();
}

function sendToBackend() {
  console.log("Send this text to backend:", finalTranscript.value);

  // Example for future Socket.io
  // socket.emit("candidate_answer", finalTranscript.value)

  alert("Transcript sent! Check console.");
}

//------------------------------------------- TTS --------------------------------------------------//
function loadVoices() {
  voices.value = synth.getVoices();
}

function speak() {
  if (!text.value) return;

  // Stop any current speech
  synth.cancel();

  const utterance = new SpeechSynthesisUtterance(text.value);

  const selectedVoice = voices.value[selectedVoiceIndex.value];
  if (selectedVoice) {
    utterance.voice = selectedVoice;
  }

  utterance.rate = 1; // speed (0.5 - 2)
  utterance.pitch = 1; // tone (0 - 2)
  utterance.volume = 1; // volume (0 - 1)

  synth.speak(utterance);
}

function stop() {
  synth.cancel();
}

// ---------------------------------------------- Lifecycle Hooks ----------------------------------------------//
onMounted(() => {
  // Initialize Speech Recognition
  const SpeechRecognition =
    window.SpeechRecognition || window.webkitSpeechRecognition;

  if (!SpeechRecognition) {
    alert("Web Speech API not supported in this browser (use Chrome).");
    return;
  }
  console.log(
    "SpeechRecognition supported, initializing...",
    SpeechRecognition,
  );

  recognition = new SpeechRecognition();
  recognition.continuous = true;
  recognition.interimResults = true;
  recognition.lang = "en-US";

  recognition.onstart = () => {
    isListening.value = true;
  };

  recognition.onend = () => {
    isListening.value = false;
  };

  recognition.onerror = (event) => {
    console.error("Speech error:", event.error);
  };

  recognition.onresult = (event) => {
    let interim = "";
    let final = finalTranscript.value;

    for (let i = event.resultIndex; i < event.results.length; i++) {
      const transcript = event.results[i][0].transcript;

      if (event.results[i].isFinal) {
        final += transcript + " ";
      } else {
        interim += transcript;
      }
    }

    finalTranscript.value = final;
    interimTranscript.value = interim;
  };

  //tts
  loadVoices();

  // Voices load asynchronously in some browsers
  speechSynthesis.onvoiceschanged = loadVoices;
});

onBeforeUnmount(() => {
  if (recognition) recognition.stop();
});
</script>

<template>
  <!------------------------- Transcribe ----------------->
  <div class="w-full max-w-xl mx-auto mt-10 border rounded-lg p-6">
    <h2 class="text-center text-xl">AI Interview - Speech Input</h2>

    <div class="status">
      Status:
      <span :class="{ active: isListening }">
        {{ isListening ? "Listening..." : "Idle" }}
      </span>
    </div>

    <div class="transcript-box">
      <h4>Live Transcript</h4>
      <p>{{ interimTranscript }}</p>
    </div>

    <div class="transcript-box final">
      <h4>Final Transcript</h4>
      <p>{{ finalTranscript }}</p>
    </div>

    <div class="buttons">
      <button
        @click="startListening"
        :disabled="isListening"
        class="rounded px-3 py-1 border cursor-pointer hover:bg-gray-100 hover:scale-95"
      >
        Start Speaking
      </button>

      <button
        @click="stopListening"
        :disabled="!isListening"
        class="rounded px-3 py-1 border cursor-pointer hover:bg-gray-100 hover:scale-95"
      >
        Stop
      </button>
    </div>
  </div>

  <!-- TTS Controls -->
  <div
    class="w-full max-w-xl mx-auto mt-10 border rounded-lg p-6 flex flex-col gap-4"
  >
    <h2 class="text-center text-xl">Text to Speech Demo</h2>

    <textarea
      class="border w-full rounded p-3"
      v-model="text"
      placeholder="Enter text to speak"
      rows="4"
    ></textarea>
    <p class="-mt-4 text-sm">Enter any text above to hear it spoken</p>

    <div class="flex flex-row gap-6">
      <button
        @click="speak"
        class="rounded px-3 py-1 border cursor-pointer hover:bg-gray-100 hover:scale-95"
      >
        Speak
      </button>
      <button
        @click="stop"
        class="rounded px-3 py-1 border cursor-pointer hover:bg-gray-100 hover:scale-95"
      >
        Stop
      </button>
    </div>

    <div class="flex gap-4">
      <label>Voice:</label>
      <select v-model="selectedVoiceIndex" class="cursor-pointer">
        <option v-for="(voice, index) in voices" :key="index" :value="index">
          {{ voice.name }} ({{ voice.lang }})
        </option>
      </select>
    </div>
  </div>
</template>

<style>
.editor-content * {
  all: revert;
}

.container {
  max-width: 600px;
  margin: 40px auto;
  font-family: Arial, sans-serif;
}

.status {
  margin-bottom: 10px;
}

.status .active {
  color: green;
  font-weight: bold;
}

.transcript-box {
  border: 1px solid #ccc;
  padding: 10px;
  min-height: 60px;
  margin-bottom: 10px;
  background: #f9f9f9;
}

.transcript-box.final {
  background: #eef7ff;
}

.buttons button {
  margin-right: 10px;
  padding: 8px 12px;
  cursor: pointer;
}
</style>
