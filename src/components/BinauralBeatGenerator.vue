<template>
  <div class="binaural-generator">
    <header class="app-header">
      <h1>Binaural Beat Generator</h1>
      <p>Create programmable binaural beats for meditation and focus</p>
    </header>

    <div class="controls-container">
      <!-- Frequency Controls -->
      <div class="frequency-controls">
        <h2>Frequency Settings</h2>

        <div class="frequency-inputs">
          <div class="frequency-input">
            <label for="leftFreq">Left Ear (Hz)</label>
            <input
              id="leftFreq"
              v-model.number="leftFrequency"
              type="number"
              min="20"
              max="2000"
              step="0.1"
              @input="updateFrequencies"
            />
            <span class="hz-label">Hz</span>
          </div>

          <div class="frequency-input">
            <label for="rightFreq">Right Ear (Hz)</label>
            <input
              id="rightFreq"
              v-model.number="rightFrequency"
              type="number"
              min="20"
              max="2000"
              step="0.1"
              @input="updateFrequencies"
            />
            <span class="hz-label">Hz</span>
          </div>
        </div>

        <!-- Frequency Data Display -->
        <div class="frequency-data">
          <div class="data-item">
            <span class="label">Binaural Beat:</span>
            <span class="value">{{ binauralBeatFrequency.toFixed(1) }} Hz</span>
          </div>
          <div class="data-item">
            <span class="label">Beat Type:</span>
            <span class="value">{{ beatType }}</span>
          </div>
          <div class="data-item">
            <span class="label">Carrier Frequency:</span>
            <span class="value">{{ carrierFrequency.toFixed(1) }} Hz</span>
          </div>
        </div>
      </div>

      <!-- Audio Player Controls -->
      <div class="player-controls">
        <h2>Audio Controls</h2>

        <div class="playback-controls">
          <button
            @click="togglePlayback"
            :class="['play-btn', { active: isPlaying }]"
          >
            {{ isPlaying ? "⏸️ Pause" : "▶️ Play" }}
          </button>

          <div class="volume-control">
            <label for="volume">Volume:</label>
            <input
              id="volume"
              v-model.number="volume"
              type="range"
              min="0"
              max="100"
              step="1"
              @input="updateVolume"
            />
            <span class="volume-value">{{ volume }}%</span>
          </div>
        </div>

        <!-- Noise Controls -->
        <div class="noise-controls">
          <h3>Background Noise</h3>

          <div class="noise-options">
            <div class="noise-option">
              <input
                id="whiteNoise"
                v-model="whiteNoiseEnabled"
                type="checkbox"
                @change="toggleWhiteNoise"
              />
              <label for="whiteNoise">White Noise</label>
              <input
                v-if="whiteNoiseEnabled"
                v-model.number="whiteNoiseVolume"
                type="range"
                min="0"
                max="50"
                step="1"
                @input="updateWhiteNoiseVolume"
                class="noise-volume"
              />
            </div>

            <div class="noise-option">
              <input
                id="pinkNoise"
                v-model="pinkNoiseEnabled"
                type="checkbox"
                @change="togglePinkNoise"
              />
              <label for="pinkNoise">Pink Noise</label>
              <input
                v-if="pinkNoiseEnabled"
                v-model.number="pinkNoiseVolume"
                type="range"
                min="0"
                max="50"
                step="1"
                @input="updatePinkNoiseVolume"
                class="noise-volume"
              />
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Status Display -->
    <div class="status-display">
      <div class="status-item">
        <span class="status-label">Status:</span>
        <span :class="['status-value', { playing: isPlaying }]">
          {{ isPlaying ? "Playing" : "Stopped" }}
        </span>
      </div>
      <div class="status-item" v-if="isPlaying">
        <span class="status-label">Duration:</span>
        <span class="status-value">{{ formatTime(playTime) }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue"

// Reactive data
const leftFrequency = ref(440)
const rightFrequency = ref(445)
const volume = ref(50)
const isPlaying = ref(false)
const whiteNoiseEnabled = ref(false)
const pinkNoiseEnabled = ref(false)
const whiteNoiseVolume = ref(20)
const pinkNoiseVolume = ref(20)
const playTime = ref(0)

// Audio context and nodes
let audioContext = null
let leftOscillator = null
let rightOscillator = null
let whiteNoiseNode = null
let pinkNoiseNode = null
let gainNode = null
let whiteNoiseGain = null
let pinkNoiseGain = null
let merger = null
let playTimer = null

// Computed properties
const binauralBeatFrequency = computed(() => {
  return Math.abs(rightFrequency.value - leftFrequency.value)
})

const carrierFrequency = computed(() => {
  return (leftFrequency.value + rightFrequency.value) / 2
})

const beatType = computed(() => {
  const diff = binauralBeatFrequency.value
  if (diff < 4) return "Delta (0.5-4 Hz) - Deep Sleep"
  if (diff < 8) return "Theta (4-8 Hz) - Meditation"
  if (diff < 13) return "Alpha (8-13 Hz) - Relaxation"
  if (diff < 30) return "Beta (13-30 Hz) - Focus"
  return "Gamma (30+ Hz) - High Focus"
})

// Methods
const initAudioContext = () => {
  if (!audioContext) {
    audioContext = new (window.AudioContext || window.webkitAudioContext)()

    // Create gain node for master volume
    gainNode = audioContext.createGain()
    gainNode.connect(audioContext.destination)

    // Create stereo merger for binaural audio
    merger = audioContext.createChannelMerger(2)
    merger.connect(gainNode)

    // Create noise gain nodes
    whiteNoiseGain = audioContext.createGain()
    whiteNoiseGain.gain.value = 0
    whiteNoiseGain.connect(gainNode)

    pinkNoiseGain = audioContext.createGain()
    pinkNoiseGain.gain.value = 0
    pinkNoiseGain.connect(gainNode)
  }
}

const createOscillators = () => {
  if (!audioContext) return

  // Left ear oscillator
  leftOscillator = audioContext.createOscillator()
  leftOscillator.frequency.value = leftFrequency.value
  leftOscillator.type = "sine"
  leftOscillator.connect(merger, 0, 0) // Connect to left channel

  // Right ear oscillator
  rightOscillator = audioContext.createOscillator()
  rightOscillator.frequency.value = rightFrequency.value
  rightOscillator.type = "sine"
  rightOscillator.connect(merger, 0, 1) // Connect to right channel
}

const createWhiteNoise = () => {
  if (!audioContext) return

  const bufferSize = 2 * audioContext.sampleRate
  const noiseBuffer = audioContext.createBuffer(
    2,
    bufferSize,
    audioContext.sampleRate
  )

  for (let channel = 0; channel < noiseBuffer.numberOfChannels; channel++) {
    const output = noiseBuffer.getChannelData(channel)
    for (let i = 0; i < bufferSize; i++) {
      output[i] = Math.random() * 2 - 1
    }
  }

  whiteNoiseNode = audioContext.createBufferSource()
  whiteNoiseNode.buffer = noiseBuffer
  whiteNoiseNode.loop = true
  whiteNoiseNode.connect(whiteNoiseGain)
}

const createPinkNoise = () => {
  if (!audioContext) return

  const bufferSize = 2 * audioContext.sampleRate
  const noiseBuffer = audioContext.createBuffer(
    2,
    bufferSize,
    audioContext.sampleRate
  )

  for (let channel = 0; channel < noiseBuffer.numberOfChannels; channel++) {
    const output = noiseBuffer.getChannelData(channel)
    let b0 = 0,
      b1 = 0,
      b2 = 0,
      b3 = 0,
      b4 = 0,
      b5 = 0,
      b6 = 0

    for (let i = 0; i < bufferSize; i++) {
      const white = Math.random() * 2 - 1
      b0 = 0.99886 * b0 + white * 0.0555179
      b1 = 0.99332 * b1 + white * 0.0750759
      b2 = 0.969 * b2 + white * 0.153852
      b3 = 0.8665 * b3 + white * 0.3104856
      b4 = 0.55 * b4 + white * 0.5329522
      b5 = -0.7616 * b5 - white * 0.016898
      output[i] = b0 + b1 + b2 + b3 + b4 + b5 + b6 + white * 0.5362
      output[i] *= 0.11 // Gain compensation
      b6 = white * 0.115926
    }
  }

  pinkNoiseNode = audioContext.createBufferSource()
  pinkNoiseNode.buffer = noiseBuffer
  pinkNoiseNode.loop = true
  pinkNoiseNode.connect(pinkNoiseGain)
}

const togglePlayback = async () => {
  if (!isPlaying.value) {
    await startPlayback()
  } else {
    stopPlayback()
  }
}

const startPlayback = async () => {
  try {
    initAudioContext()

    if (audioContext.state === "suspended") {
      await audioContext.resume()
    }

    createOscillators()

    if (whiteNoiseEnabled.value) {
      createWhiteNoise()
      whiteNoiseNode.start()
    }

    if (pinkNoiseEnabled.value) {
      createPinkNoise()
      pinkNoiseNode.start()
    }

    leftOscillator.start()
    rightOscillator.start()

    isPlaying.value = true
    playTime.value = 0
    playTimer = setInterval(() => {
      playTime.value++
    }, 1000)
  } catch (error) {
    console.error("Error starting playback:", error)
  }
}

const stopPlayback = () => {
  if (leftOscillator) {
    leftOscillator.stop()
    leftOscillator = null
  }

  if (rightOscillator) {
    rightOscillator.stop()
    rightOscillator = null
  }

  if (whiteNoiseNode) {
    whiteNoiseNode.stop()
    whiteNoiseNode = null
  }

  if (pinkNoiseNode) {
    pinkNoiseNode.stop()
    pinkNoiseNode = null
  }

  if (playTimer) {
    clearInterval(playTimer)
    playTimer = null
  }

  isPlaying.value = false
}

const updateFrequencies = () => {
  if (leftOscillator) {
    leftOscillator.frequency.value = leftFrequency.value
  }
  if (rightOscillator) {
    rightOscillator.frequency.value = rightFrequency.value
  }
}

const updateVolume = () => {
  if (gainNode) {
    gainNode.gain.value = volume.value / 100
  }
}

const toggleWhiteNoise = () => {
  if (isPlaying.value) {
    if (whiteNoiseEnabled.value && !whiteNoiseNode) {
      createWhiteNoise()
      whiteNoiseNode.start()
    } else if (!whiteNoiseEnabled.value && whiteNoiseNode) {
      whiteNoiseNode.stop()
      whiteNoiseNode = null
    }
  }
  updateWhiteNoiseVolume()
}

const togglePinkNoise = () => {
  if (isPlaying.value) {
    if (pinkNoiseEnabled.value && !pinkNoiseNode) {
      createPinkNoise()
      pinkNoiseNode.start()
    } else if (!pinkNoiseEnabled.value && pinkNoiseNode) {
      pinkNoiseNode.stop()
      pinkNoiseNode = null
    }
  }
  updatePinkNoiseVolume()
}

const updateWhiteNoiseVolume = () => {
  if (whiteNoiseGain) {
    whiteNoiseGain.gain.value = whiteNoiseEnabled.value
      ? whiteNoiseVolume.value / 100
      : 0
  }
}

const updatePinkNoiseVolume = () => {
  if (pinkNoiseGain) {
    pinkNoiseGain.gain.value = pinkNoiseEnabled.value
      ? pinkNoiseVolume.value / 100
      : 0
  }
}

const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}:${secs.toString().padStart(2, "0")}`
}

// Initialize volume
onMounted(() => {
  updateVolume()
})

// Cleanup on unmount
onUnmounted(() => {
  if (isPlaying.value) {
    stopPlayback()
  }
  if (audioContext) {
    audioContext.close()
  }
})
</script>

<style scoped>
.binaural-generator {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.app-header {
  text-align: center;
  margin-bottom: 30px;
}

.app-header h1 {
  color: #2c3e50;
  margin-bottom: 10px;
}

.app-header p {
  color: #7f8c8d;
  font-size: 16px;
}

.controls-container {
  display: grid;
  gap: 30px;
  margin-bottom: 30px;
}

.frequency-controls,
.player-controls {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e9ecef;
}

.frequency-controls h2,
.player-controls h2,
.noise-controls h3 {
  margin-top: 0;
  color: #2c3e50;
}

.frequency-inputs {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 20px;
}

.frequency-input {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.frequency-input label {
  font-weight: 600;
  color: #495057;
}

.frequency-input input[type="number"] {
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 16px;
}

.hz-label {
  font-size: 14px;
  color: #6c757d;
}

.frequency-data {
  background: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
}

.data-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.data-item:last-child {
  margin-bottom: 0;
}

.label {
  font-weight: 600;
  color: #495057;
}

.value {
  color: #007bff;
  font-weight: 600;
}

.playback-controls {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.play-btn {
  padding: 12px 24px;
  font-size: 16px;
  font-weight: 600;
  border: none;
  border-radius: 6px;
  background: #007bff;
  color: white;
  cursor: pointer;
  transition: background-color 0.2s;
}

.play-btn:hover {
  background: #0056b3;
}

.play-btn.active {
  background: #dc3545;
}

.play-btn.active:hover {
  background: #c82333;
}

.volume-control {
  display: flex;
  align-items: center;
  gap: 10px;
}

.volume-control input[type="range"] {
  width: 120px;
}

.volume-value {
  min-width: 40px;
  font-weight: 600;
  color: #495057;
}

.noise-controls {
  background: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
}

.noise-options {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.noise-option {
  display: flex;
  align-items: center;
  gap: 10px;
}

.noise-option input[type="checkbox"] {
  width: 18px;
  height: 18px;
}

.noise-option label {
  font-weight: 600;
  color: #495057;
  min-width: 80px;
}

.noise-volume {
  width: 100px;
}

.status-display {
  background: #e9ecef;
  padding: 15px;
  border-radius: 4px;
  display: flex;
  gap: 30px;
  flex-wrap: wrap;
}

.status-item {
  display: flex;
  gap: 8px;
}

.status-label {
  font-weight: 600;
  color: #495057;
}

.status-value {
  color: #6c757d;
}

.status-value.playing {
  color: #28a745;
  font-weight: 600;
}

@media (max-width: 768px) {
  .frequency-inputs {
    grid-template-columns: 1fr;
  }

  .playback-controls {
    flex-direction: column;
    align-items: stretch;
  }

  .volume-control {
    justify-content: space-between;
  }

  .status-display {
    flex-direction: column;
    gap: 10px;
  }
}
</style>
