<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

//------------------------------------------------- State & Refs -------------------------------------------------//

// for speech recognition
const isListening = ref(false);
const interimTranscript = ref("");
const finalTranscript = ref("");
let recognition = null;

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
});

onBeforeUnmount(() => {
  if (recognition) recognition.stop();
});
</script>

<template>
  <!-- Transcribe -->
  <div class="container">
    <h2>AI Interview - Speech Input</h2>

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
      <button @click="startListening" :disabled="isListening">
        Start Speaking
      </button>

      <button @click="stopListening" :disabled="!isListening">Stop</button>

      <button @click="sendToBackend" :disabled="!finalTranscript">
        Send to Backend
      </button>
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
