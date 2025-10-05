<template>
  <div class="binaural-generator" :class="{ 'dark-mode': isDarkMode }">
    <header class="app-header">
      <div class="header-content">
        <div class="title-section">
          <h1>Binaural Beat Generator</h1>
          <p>Create programmable binaural beats for meditation and focus</p>
        </div>
        <div class="theme-toggle">
          <button @click="toggleDarkMode" class="theme-toggle-btn">
            {{ isDarkMode ? "☀️" : "🌙" }}
          </button>
        </div>
      </div>
    </header>

    <div class="controls-container">
      <!-- Frequency Controls -->
      <div class="frequency-controls">
        <h2>Frequency Settings</h2>

        <!-- Frequency Input Mode Toggle -->
        <div class="frequency-mode-toggle">
          <div class="toggle-buttons">
            <button
              @click="frequencyMode = 'leftright'"
              :class="['mode-btn', { active: frequencyMode === 'leftright' }]"
            >
              Left/Right Mode
            </button>
            <button
              @click="frequencyMode = 'carrier'"
              :class="['mode-btn', { active: frequencyMode === 'carrier' }]"
            >
              Carrier + Beat Mode
            </button>
          </div>
        </div>

        <!-- Left/Right Frequency Inputs -->
        <div v-if="frequencyMode === 'leftright'" class="frequency-inputs">
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

        <!-- Carrier + Beat Frequency Inputs -->
        <div v-if="frequencyMode === 'carrier'" class="frequency-inputs">
          <div class="frequency-input">
            <label for="carrierFreq">
              Carrier Frequency (Hz)
              <span class="frequency-hint">Recommended: 200-800 Hz</span>
            </label>
            <input
              id="carrierFreq"
              v-model.number="manualCarrierFreq"
              type="number"
              min="50"
              max="1500"
              step="0.1"
              @input="updateFromCarrier"
            />
            <span class="hz-label">Hz</span>
            <div v-if="carrierWarning" class="frequency-warning">
              {{ carrierWarning }}
            </div>
          </div>

          <div class="frequency-input">
            <label for="beatFreq">
              Binaural Beat (Hz)
              <span class="frequency-hint">Difference between ears</span>
            </label>
            <input
              id="beatFreq"
              v-model.number="manualBeatFreq"
              type="number"
              min="0.1"
              max="50"
              step="0.1"
              @input="updateFromCarrier"
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

      <!-- Session Controls -->
      <div class="session-controls">
        <h2>Session Settings</h2>

        <!-- Preset Selection -->
        <div class="preset-selection">
          <label for="presetSelect">Choose Preset:</label>
          <select
            id="presetSelect"
            v-model="selectedPreset"
            @change="loadPreset"
            :disabled="isPlaying"
          >
            <option value="manual">Manual Mode</option>
            <option
              v-for="preset in presets"
              :key="preset.id"
              :value="preset.id"
            >
              {{ preset.name }}
            </option>
          </select>
          <button
            @click="showPresetEditor = true"
            class="edit-preset-btn"
            :disabled="isPlaying"
          >
            Edit Presets
          </button>
        </div>

        <!-- Session Timer -->
        <div class="session-timer">
          <div class="timer-controls">
            <label for="sessionDuration">Session Duration:</label>
            <select
              id="sessionDuration"
              v-model="sessionDuration"
              :disabled="isPlaying"
            >
              <option value="0">Continuous</option>
              <option value="300">5 minutes</option>
              <option value="600">10 minutes</option>
              <option value="900">15 minutes</option>
              <option value="1200">20 minutes</option>
              <option value="1800">30 minutes</option>
              <option value="2400">40 minutes</option>
              <option value="2700">45 minutes</option>
              <option value="3600">60 minutes</option>
              <option value="5400">90 minutes</option>
            </select>
          </div>

          <div class="timer-display" v-if="sessionDuration > 0">
            <div class="progress-bar">
              <div
                class="progress-fill"
                :style="{ width: sessionProgress + '%' }"
              ></div>
            </div>
            <div class="time-remaining">
              {{ formatTime(timeRemaining) }} remaining
            </div>
          </div>
        </div>

        <!-- Current Session Info -->
        <div
          class="session-info"
          v-if="selectedPreset !== 'manual' && currentProgram"
        >
          <h4>Current Program: {{ currentProgram.name }}</h4>
          <div class="program-stage" v-if="currentStage">
            <span class="stage-label">Stage {{ currentStageIndex + 1 }}:</span>
            <span class="stage-description">{{
              currentStage.description
            }}</span>
            <div class="stage-progress">
              <div class="stage-time">
                {{ formatTime(stageTimeElapsed) }} /
                {{ formatTime(currentStage.duration) }}
              </div>
            </div>
          </div>
        </div>

        <!-- Session Plan View -->
        <div
          class="session-plan"
          v-if="selectedPreset !== 'manual' && currentProgram"
        >
          <div class="plan-header">
            <h4>Session Plan</h4>
            <button
              @click="showSessionPlan = !showSessionPlan"
              class="toggle-plan-btn"
              :class="{ expanded: showSessionPlan }"
            >
              {{ showSessionPlan ? "▼ Hide Plan" : "▶ View Full Plan" }}
            </button>
          </div>

          <div v-if="showSessionPlan" class="plan-content">
            <!-- Session Overview -->
            <div class="session-overview">
              <div class="overview-item">
                <span class="overview-label">Total Duration:</span>
                <span class="overview-value">{{
                  formatTime(currentProgram.duration)
                }}</span>
              </div>
              <div class="overview-item">
                <span class="overview-label">Stages:</span>
                <span class="overview-value">{{
                  currentProgram.stages.length
                }}</span>
              </div>
              <div class="overview-item">
                <span class="overview-label">Type:</span>
                <span class="overview-value">{{
                  getProgramType(currentProgram)
                }}</span>
              </div>
            </div>

            <!-- Visual Progress Map -->
            <div class="progress-map">
              <div class="map-title">Session Progress Map</div>
              <div class="timeline">
                <div
                  v-for="(stage, index) in currentProgram.stages"
                  :key="index"
                  class="timeline-segment"
                  :class="{
                    active: index === currentStageIndex && isPlaying,
                    completed: index < currentStageIndex && isPlaying,
                    current: index === currentStageIndex,
                  }"
                  :style="{ width: getStageWidthPercent(stage) + '%' }"
                >
                  <div class="segment-label">{{ stage.description }}</div>
                  <div class="segment-time">
                    {{ formatTime(stage.duration) }}
                  </div>
                </div>
              </div>
            </div>

            <!-- Detailed Stage Information -->
            <div class="stages-detail">
              <div class="detail-title">Stage Details</div>
              <div
                v-for="(stage, index) in currentProgram.stages"
                :key="index"
                class="stage-detail"
                :class="{
                  active: index === currentStageIndex && isPlaying,
                  completed: index < currentStageIndex && isPlaying,
                }"
              >
                <div class="stage-header-detail">
                  <div class="stage-number">{{ index + 1 }}</div>
                  <div class="stage-info">
                    <div class="stage-name">{{ stage.description }}</div>
                    <div class="stage-duration">
                      {{ formatTime(stage.duration) }}
                    </div>
                  </div>
                  <div class="stage-status">
                    <span
                      v-if="index < currentStageIndex && isPlaying"
                      class="status-completed"
                      >✓</span
                    >
                    <span
                      v-else-if="index === currentStageIndex && isPlaying"
                      class="status-active"
                      >►</span
                    >
                    <span v-else class="status-pending">○</span>
                  </div>
                </div>
                <div class="stage-frequencies">
                  <div class="freq-info">
                    <span class="freq-label">Left:</span>
                    <span class="freq-value">{{ stage.leftFreq }} Hz</span>
                  </div>
                  <div class="freq-info">
                    <span class="freq-label">Right:</span>
                    <span class="freq-value">{{ stage.rightFreq }} Hz</span>
                  </div>
                  <div class="freq-info">
                    <span class="freq-label">Beat:</span>
                    <span class="freq-value"
                      >{{
                        Math.abs(stage.rightFreq - stage.leftFreq).toFixed(1)
                      }}
                      Hz</span
                    >
                  </div>
                  <div class="freq-info">
                    <span class="freq-label">Type:</span>
                    <span class="freq-value">{{
                      getBeatType(Math.abs(stage.rightFreq - stage.leftFreq))
                    }}</span>
                  </div>
                  <div class="freq-info">
                    <span class="freq-label">Volume:</span>
                    <span class="freq-value">{{ stage.volume }}%</span>
                  </div>
                </div>
              </div>
            </div>
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
            {{ isPlaying ? "Pause" : "Play" }}
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

            <div class="noise-option">
              <input
                id="brownNoise"
                v-model="brownNoiseEnabled"
                type="checkbox"
                @change="toggleBrownNoise"
              />
              <label for="brownNoise">Brown Noise</label>
              <input
                v-if="brownNoiseEnabled"
                v-model.number="brownNoiseVolume"
                type="range"
                min="0"
                max="50"
                step="1"
                @input="updateBrownNoiseVolume"
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
      <div class="status-item" v-if="sessionDuration > 0">
        <span class="status-label">Session Progress:</span>
        <span class="status-value">{{ Math.round(sessionProgress) }}%</span>
      </div>
    </div>

    <!-- Preset Editor Modal -->
    <div
      v-if="showPresetEditor"
      class="modal-overlay"
      @click="closePresetEditor"
    >
      <div class="preset-editor-modal" @click.stop>
        <div class="modal-header">
          <h3>Edit Presets</h3>
          <button @click="closePresetEditor" class="close-btn">×</button>
        </div>

        <div class="modal-content">
          <div class="preset-list">
            <h4>Existing Presets</h4>
            <div v-for="preset in presets" :key="preset.id" class="preset-item">
              <div class="preset-info">
                <strong>{{ preset.name }}</strong>
                <span class="preset-description">{{ preset.description }}</span>
                <span class="preset-duration">{{
                  formatTime(preset.duration)
                }}</span>
              </div>
              <div class="preset-actions">
                <button @click="editPreset(preset)" class="edit-btn">
                  Edit
                </button>
                <button @click="deletePreset(preset.id)" class="delete-btn">
                  Delete
                </button>
              </div>
            </div>
          </div>

          <div class="create-preset">
            <h4>{{ editingPreset ? "Edit Preset" : "Create New Preset" }}</h4>
            <div class="preset-form">
              <div class="form-group">
                <label>Name:</label>
                <input
                  v-model="newPreset.name"
                  type="text"
                  placeholder="Preset name"
                />
              </div>
              <div class="form-group">
                <label>Description:</label>
                <input
                  v-model="newPreset.description"
                  type="text"
                  placeholder="Brief description"
                />
              </div>

              <div class="stages-section">
                <h5>Program Stages</h5>
                <div
                  v-for="(stage, index) in newPreset.stages"
                  :key="index"
                  class="stage-editor"
                >
                  <div class="stage-header">
                    <h6>Stage {{ index + 1 }}</h6>
                    <button
                      @click="removeStage(index)"
                      class="remove-stage-btn"
                    >
                      Remove
                    </button>
                  </div>
                  <div class="stage-controls">
                    <div class="control-group">
                      <label>Description:</label>
                      <input
                        v-model="stage.description"
                        type="text"
                        placeholder="Stage description"
                      />
                    </div>
                    <div class="control-group">
                      <label>Duration (minutes):</label>
                      <input
                        v-model.number="stage.durationMinutes"
                        type="number"
                        min="1"
                        max="60"
                      />
                    </div>
                    <div class="control-group">
                      <label>Left Frequency (Hz):</label>
                      <input
                        v-model.number="stage.leftFreq"
                        type="number"
                        min="20"
                        max="2000"
                        step="0.1"
                      />
                    </div>
                    <div class="control-group">
                      <label>Right Frequency (Hz):</label>
                      <input
                        v-model.number="stage.rightFreq"
                        type="number"
                        min="20"
                        max="2000"
                        step="0.1"
                      />
                    </div>
                    <div class="control-group">
                      <label>Volume (%):</label>
                      <input
                        v-model.number="stage.volume"
                        type="number"
                        min="0"
                        max="100"
                      />
                    </div>
                  </div>
                </div>

                <button @click="addStage" class="add-stage-btn">
                  Add Stage
                </button>
              </div>

              <div class="form-actions">
                <button @click="savePreset" class="save-btn">
                  {{ editingPreset ? "Update" : "Create" }} Preset
                </button>
                <button @click="cancelEdit" class="cancel-btn">Cancel</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from "vue"

// Reactive data
const leftFrequency = ref(440)
const rightFrequency = ref(445)
const volume = ref(50)
const isPlaying = ref(false)

// Carrier frequency mode
const frequencyMode = ref("leftright") // 'leftright' or 'carrier'
const manualCarrierFreq = ref(442.5) // Manual carrier frequency
const manualBeatFreq = ref(5) // Manual beat frequency
const whiteNoiseEnabled = ref(false)
const pinkNoiseEnabled = ref(false)
const whiteNoiseVolume = ref(20)
const pinkNoiseVolume = ref(20)
const brownNoiseEnabled = ref(false)
const brownNoiseVolume = ref(20)

// Dark mode
const isDarkMode = ref(false)

const playTime = ref(0)

// Session and preset data
const selectedPreset = ref("manual")
const sessionDuration = ref(0)
const sessionStartTime = ref(0)
const showPresetEditor = ref(false)
const showSessionPlan = ref(false)
const editingPreset = ref(null)
const currentProgram = ref(null)
const currentStageIndex = ref(0)
const stageStartTime = ref(0)

// Default presets
const presets = ref([
  {
    id: "focus",
    name: "Deep Focus",
    description: "20-minute focus enhancement program",
    duration: 1200,
    stages: [
      {
        description: "Preparation",
        duration: 300,
        leftFreq: 100,
        rightFreq: 110,
        volume: 40,
        durationMinutes: 5,
      },
      {
        description: "Focus Building",
        duration: 600,
        leftFreq: 120,
        rightFreq: 135,
        volume: 50,
        durationMinutes: 10,
      },
      {
        description: "Deep Focus",
        duration: 300,
        leftFreq: 140,
        rightFreq: 158,
        volume: 45,
        durationMinutes: 5,
      },
    ],
  },
  {
    id: "meditation",
    name: "Meditation",
    description: "15-minute meditation program",
    duration: 900,
    stages: [
      {
        description: "Relaxation",
        duration: 300,
        leftFreq: 200,
        rightFreq: 206,
        volume: 35,
        durationMinutes: 5,
      },
      {
        description: "Deep Meditation",
        duration: 600,
        leftFreq: 150,
        rightFreq: 154,
        volume: 40,
        durationMinutes: 10,
      },
    ],
  },
  {
    id: "sleep",
    name: "Sleep Preparation",
    description: "30-minute sleep induction program",
    duration: 1800,
    stages: [
      {
        description: "Wind Down",
        duration: 600,
        leftFreq: 300,
        rightFreq: 308,
        volume: 30,
        durationMinutes: 10,
      },
      {
        description: "Theta Waves",
        duration: 600,
        leftFreq: 200,
        rightFreq: 206,
        volume: 25,
        durationMinutes: 10,
      },
      {
        description: "Delta Waves",
        duration: 600,
        leftFreq: 100,
        rightFreq: 103,
        volume: 20,
        durationMinutes: 10,
      },
    ],
  },
  {
    id: "deep-sleep",
    name: "Deep Sleep",
    description: "90-minute deep sleep program with delta waves (0.5-2 Hz)",
    duration: 5400,
    stages: [
      {
        description: "Initial Relaxation",
        duration: 600,
        leftFreq: 250,
        rightFreq: 254,
        volume: 25,
        durationMinutes: 10,
      },
      {
        description: "Theta Transition",
        duration: 600,
        leftFreq: 200,
        rightFreq: 206,
        volume: 22,
        durationMinutes: 10,
      },
      {
        description: "Light Delta",
        duration: 1200,
        leftFreq: 150,
        rightFreq: 152,
        volume: 20,
        durationMinutes: 20,
      },
      {
        description: "Deep Delta Sleep",
        duration: 3000,
        leftFreq: 100,
        rightFreq: 101.5,
        volume: 18,
        durationMinutes: 50,
      },
    ],
  },
  {
    id: "power-nap",
    name: "Power Nap",
    description: "20-minute refreshing power nap with 1.5 Hz delta waves",
    duration: 1200,
    stages: [
      {
        description: "Quick Relaxation",
        duration: 300,
        leftFreq: 200,
        rightFreq: 204,
        volume: 30,
        durationMinutes: 5,
      },
      {
        description: "Delta Rest",
        duration: 600,
        leftFreq: 120,
        rightFreq: 121.5,
        volume: 25,
        durationMinutes: 10,
      },
      {
        description: "Gentle Wake Prep",
        duration: 300,
        leftFreq: 150,
        rightFreq: 153,
        volume: 28,
        durationMinutes: 5,
      },
    ],
  },
  {
    id: "cellular-healing",
    name: "Cellular Healing",
    description: "60-minute healing session with 1-2 Hz delta frequencies",
    duration: 3600,
    stages: [
      {
        description: "Preparation",
        duration: 600,
        leftFreq: 180,
        rightFreq: 184,
        volume: 25,
        durationMinutes: 10,
      },
      {
        description: "Healing Frequencies",
        duration: 2400,
        leftFreq: 110,
        rightFreq: 111.5,
        volume: 22,
        durationMinutes: 40,
      },
      {
        description: "Integration",
        duration: 600,
        leftFreq: 140,
        rightFreq: 142,
        volume: 20,
        durationMinutes: 10,
      },
    ],
  },
  {
    id: "lucid-sleep-primer",
    name: "Lucid Sleep Primer",
    description: "45-minute cycle: Alpha → Theta → Delta for lucid dreaming",
    duration: 2700,
    stages: [
      {
        description: "Alpha Relaxation",
        duration: 600,
        leftFreq: 250,
        rightFreq: 260,
        volume: 30,
        durationMinutes: 10,
      },
      {
        description: "Alpha-Theta Bridge",
        duration: 600,
        leftFreq: 200,
        rightFreq: 208,
        volume: 28,
        durationMinutes: 10,
      },
      {
        description: "Theta Dream State",
        duration: 900,
        leftFreq: 150,
        rightFreq: 156,
        volume: 25,
        durationMinutes: 15,
      },
      {
        description: "Light Delta Sleep",
        duration: 600,
        leftFreq: 120,
        rightFreq: 123,
        volume: 22,
        durationMinutes: 10,
      },
    ],
  },
  {
    id: "recovery-healing",
    name: "Recovery & Healing",
    description:
      "40-minute physical recovery session with delta waves for tissue repair",
    duration: 2400,
    stages: [
      {
        description: "Initial Relaxation",
        duration: 600,
        leftFreq: 200,
        rightFreq: 206,
        volume: 25,
        durationMinutes: 10,
      },
      {
        description: "Deep Healing Phase",
        duration: 1200,
        leftFreq: 100,
        rightFreq: 102,
        volume: 20,
        durationMinutes: 20,
      },
      {
        description: "Cellular Regeneration",
        duration: 600,
        leftFreq: 80,
        rightFreq: 81.5,
        volume: 18,
        durationMinutes: 10,
      },
    ],
  },
  {
    id: "pain-relief",
    name: "Pain Relief",
    description: "30-minute natural pain management with theta waves",
    duration: 1800,
    stages: [
      {
        description: "Tension Release",
        duration: 600,
        leftFreq: 150,
        rightFreq: 156,
        volume: 30,
        durationMinutes: 10,
      },
      {
        description: "Pain Reduction",
        duration: 900,
        leftFreq: 120,
        rightFreq: 126,
        volume: 25,
        durationMinutes: 15,
      },
      {
        description: "Comfort Integration",
        duration: 300,
        leftFreq: 100,
        rightFreq: 104,
        volume: 22,
        durationMinutes: 5,
      },
    ],
  },
  {
    id: "migraine-relief",
    name: "Migraine Relief",
    description:
      "20-minute headache and migraine relief with theta frequencies",
    duration: 1200,
    stages: [
      {
        description: "Vascular Relaxation",
        duration: 300,
        leftFreq: 140,
        rightFreq: 146,
        volume: 20,
        durationMinutes: 5,
      },
      {
        description: "Pain Dissolution",
        duration: 600,
        leftFreq: 110,
        rightFreq: 116,
        volume: 18,
        durationMinutes: 10,
      },
      {
        description: "Gentle Recovery",
        duration: 300,
        leftFreq: 90,
        rightFreq: 94,
        volume: 15,
        durationMinutes: 5,
      },
    ],
  },
])

// Preset editor data
const newPreset = ref({
  name: "",
  description: "",
  stages: [
    {
      description: "",
      duration: 300,
      leftFreq: 440,
      rightFreq: 445,
      volume: 50,
      durationMinutes: 5,
    },
  ],
})

// Audio context and nodes
let audioContext = null
let leftOscillator = null
let rightOscillator = null
let whiteNoiseNode = null
let pinkNoiseNode = null
let brownNoiseNode = null
let gainNode = null
let whiteNoiseGain = null
let pinkNoiseGain = null
let brownNoiseGain = null
let merger = null
let playTimer = null

// Computed properties
const binauralBeatFrequency = computed(() => {
  return Math.abs(rightFrequency.value - leftFrequency.value)
})

const carrierFrequency = computed(() => {
  if (frequencyMode.value === "carrier") {
    return manualCarrierFreq.value
  }
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

// Carrier frequency validation
const carrierWarning = computed(() => {
  const freq = manualCarrierFreq.value
  if (freq < 100)
    return "Warning: Very low carrier frequency - may be less effective"
  if (freq > 1000)
    return "Warning: High carrier frequency - may be uncomfortable"
  if (freq < 200)
    return "Tip: 200-800 Hz range is most effective for most people"
  return ""
})

// Session computed properties
const sessionProgress = computed(() => {
  if (sessionDuration.value === 0 || !isPlaying.value) return 0
  const elapsed = playTime.value
  return Math.min((elapsed / sessionDuration.value) * 100, 100)
})

const timeRemaining = computed(() => {
  if (sessionDuration.value === 0) return 0
  return Math.max(sessionDuration.value - playTime.value, 0)
})

const currentStage = computed(() => {
  if (!currentProgram.value || !currentProgram.value.stages) return null
  return currentProgram.value.stages[currentStageIndex.value] || null
})

const stageTimeElapsed = computed(() => {
  if (!isPlaying.value || !currentStage.value) return 0
  return playTime.value - stageStartTime.value
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

    brownNoiseGain = audioContext.createGain()
    brownNoiseGain.gain.value = 0
    brownNoiseGain.connect(gainNode)
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

const createBrownNoise = () => {
  if (!audioContext) return

  const bufferSize = 2 * audioContext.sampleRate
  const noiseBuffer = audioContext.createBuffer(
    2,
    bufferSize,
    audioContext.sampleRate
  )

  for (let channel = 0; channel < noiseBuffer.numberOfChannels; channel++) {
    const output = noiseBuffer.getChannelData(channel)
    let lastOut = 0.0

    for (let i = 0; i < bufferSize; i++) {
      const white = Math.random() * 2 - 1
      output[i] = (lastOut + 0.02 * white) / 1.02
      lastOut = output[i]
      output[i] *= 3.5 // Gain compensation
    }
  }

  brownNoiseNode = audioContext.createBufferSource()
  brownNoiseNode.buffer = noiseBuffer
  brownNoiseNode.loop = true
  brownNoiseNode.connect(brownNoiseGain)
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

    if (brownNoiseEnabled.value) {
      createBrownNoise()
      brownNoiseNode.start()
    }

    leftOscillator.start()
    rightOscillator.start()

    isPlaying.value = true
    playTime.value = 0
    sessionStartTime.value = Date.now()
    stageStartTime.value = 0

    playTimer = setInterval(() => {
      playTime.value++
      checkSessionProgress()
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

  if (brownNoiseNode) {
    brownNoiseNode.stop()
    brownNoiseNode = null
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

const toggleBrownNoise = () => {
  if (isPlaying.value) {
    if (brownNoiseEnabled.value && !brownNoiseNode) {
      createBrownNoise()
      brownNoiseNode.start()
    } else if (!brownNoiseEnabled.value && brownNoiseNode) {
      brownNoiseNode.stop()
      brownNoiseNode = null
    }
  }
  updateBrownNoiseVolume()
}

const updateBrownNoiseVolume = () => {
  if (brownNoiseGain) {
    brownNoiseGain.gain.value = brownNoiseEnabled.value
      ? brownNoiseVolume.value / 100
      : 0
  }
}

// Dark mode toggle
const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value
  // Save preference to localStorage
  localStorage.setItem("binaural-dark-mode", isDarkMode.value.toString())
}

const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}:${secs.toString().padStart(2, "0")}`
}

// Session plan helper methods
const getProgramType = (program) => {
  if (!program || !program.stages) return "Unknown"

  const avgBeatFreq =
    program.stages.reduce((sum, stage) => {
      return sum + Math.abs(stage.rightFreq - stage.leftFreq)
    }, 0) / program.stages.length

  if (avgBeatFreq < 4) return "Delta Program"
  if (avgBeatFreq < 8) return "Theta Program"
  if (avgBeatFreq < 13) return "Alpha Program"
  if (avgBeatFreq < 30) return "Beta Program"
  return "Gamma Program"
}

const getStageWidthPercent = (stage) => {
  if (!currentProgram.value) return 0
  return (stage.duration / currentProgram.value.duration) * 100
}

const getBeatType = (beatFreq) => {
  if (beatFreq < 4) return "Delta"
  if (beatFreq < 8) return "Theta"
  if (beatFreq < 13) return "Alpha"
  if (beatFreq < 30) return "Beta"
  return "Gamma"
}

// Frequency mode methods
const updateFromCarrier = () => {
  if (frequencyMode.value !== "carrier") return

  const carrier = manualCarrierFreq.value
  const beat = manualBeatFreq.value

  // Calculate left and right frequencies from carrier and beat
  leftFrequency.value = carrier - beat / 2
  rightFrequency.value = carrier + beat / 2

  // Ensure frequencies stay within valid range
  if (leftFrequency.value < 20) {
    leftFrequency.value = 20
    rightFrequency.value = 20 + beat
    manualCarrierFreq.value = (leftFrequency.value + rightFrequency.value) / 2
  }
  if (rightFrequency.value > 2000) {
    rightFrequency.value = 2000
    leftFrequency.value = 2000 - beat
    manualCarrierFreq.value = (leftFrequency.value + rightFrequency.value) / 2
  }

  updateFrequencies()
}

const syncManualValues = () => {
  // Sync manual values when switching from left/right mode
  if (frequencyMode.value === "carrier") {
    manualCarrierFreq.value = (leftFrequency.value + rightFrequency.value) / 2
    manualBeatFreq.value = Math.abs(rightFrequency.value - leftFrequency.value)
  }
}

// Watch for frequency mode changes
const handleFrequencyModeChange = () => {
  if (frequencyMode.value === "carrier") {
    syncManualValues()
  }
}

// Session management methods
const loadPreset = () => {
  if (selectedPreset.value === "manual") {
    currentProgram.value = null
    sessionDuration.value = 0
    return
  }

  const preset = presets.value.find((p) => p.id === selectedPreset.value)
  if (preset) {
    currentProgram.value = preset
    sessionDuration.value = preset.duration
    currentStageIndex.value = 0

    // Load first stage settings
    if (preset.stages && preset.stages.length > 0) {
      const firstStage = preset.stages[0]
      leftFrequency.value = firstStage.leftFreq
      rightFrequency.value = firstStage.rightFreq
      volume.value = firstStage.volume
    }
  }
}

const checkSessionProgress = () => {
  if (!isPlaying.value) return

  // Check if session should end
  if (sessionDuration.value > 0 && playTime.value >= sessionDuration.value) {
    fadeOutAndStop()
    return
  }

  // Check for stage transitions in preset programs
  if (currentProgram.value && currentStage.value) {
    const stageElapsed = playTime.value - stageStartTime.value

    if (stageElapsed >= currentStage.value.duration) {
      // Move to next stage
      if (currentStageIndex.value < currentProgram.value.stages.length - 1) {
        currentStageIndex.value++
        stageStartTime.value = playTime.value

        const nextStage = currentProgram.value.stages[currentStageIndex.value]
        if (nextStage) {
          // Smooth transition to next stage
          transitionToStage(nextStage)
        }
      }
    }
  }
}

const transitionToStage = (stage) => {
  // Gradually transition frequencies and volume
  const transitionDuration = 5000 // 5 seconds
  const steps = 50
  const stepDuration = transitionDuration / steps

  const startLeft = leftFrequency.value
  const startRight = rightFrequency.value
  const startVolume = volume.value

  const targetLeft = stage.leftFreq
  const targetRight = stage.rightFreq
  const targetVolume = stage.volume

  let currentStep = 0

  const transition = setInterval(() => {
    currentStep++
    const progress = currentStep / steps

    leftFrequency.value = startLeft + (targetLeft - startLeft) * progress
    rightFrequency.value = startRight + (targetRight - startRight) * progress
    volume.value = startVolume + (targetVolume - startVolume) * progress

    updateFrequencies()
    updateVolume()

    if (currentStep >= steps) {
      clearInterval(transition)
    }
  }, stepDuration)
}

const fadeOutAndStop = () => {
  if (!gainNode) return

  const fadeTime = 3.0 // 3 second fade
  const currentVolume = gainNode.gain.value

  gainNode.gain.setValueAtTime(currentVolume, audioContext.currentTime)
  gainNode.gain.linearRampToValueAtTime(0, audioContext.currentTime + fadeTime)

  setTimeout(() => {
    stopPlayback()
    gainNode.gain.setValueAtTime(volume.value / 100, audioContext.currentTime)
  }, fadeTime * 1000)
}

// Preset editor methods
const closePresetEditor = () => {
  showPresetEditor.value = false
  cancelEdit()
}

const editPreset = (preset) => {
  editingPreset.value = preset
  newPreset.value = {
    name: preset.name,
    description: preset.description,
    stages: preset.stages.map((stage) => ({
      ...stage,
      durationMinutes: stage.duration / 60,
    })),
  }
}

const deletePreset = (presetId) => {
  if (confirm("Are you sure you want to delete this preset?")) {
    const index = presets.value.findIndex((p) => p.id === presetId)
    if (index !== -1) {
      presets.value.splice(index, 1)
    }
  }
}

const addStage = () => {
  newPreset.value.stages.push({
    description: "",
    duration: 300,
    leftFreq: 440,
    rightFreq: 445,
    volume: 50,
    durationMinutes: 5,
  })
}

const removeStage = (index) => {
  if (newPreset.value.stages.length > 1) {
    newPreset.value.stages.splice(index, 1)
  }
}

const savePreset = () => {
  if (!newPreset.value.name.trim()) {
    alert("Please enter a preset name")
    return
  }

  // Convert duration minutes to seconds and calculate total duration
  const stages = newPreset.value.stages.map((stage) => ({
    ...stage,
    duration: stage.durationMinutes * 60,
  }))

  const totalDuration = stages.reduce((sum, stage) => sum + stage.duration, 0)

  const preset = {
    id: editingPreset.value ? editingPreset.value.id : Date.now().toString(),
    name: newPreset.value.name,
    description: newPreset.value.description,
    duration: totalDuration,
    stages,
  }

  if (editingPreset.value) {
    // Update existing preset
    const index = presets.value.findIndex(
      (p) => p.id === editingPreset.value.id
    )
    if (index !== -1) {
      presets.value[index] = preset
    }
  } else {
    // Add new preset
    presets.value.push(preset)
  }

  closePresetEditor()
}

const cancelEdit = () => {
  editingPreset.value = null
  newPreset.value = {
    name: "",
    description: "",
    stages: [
      {
        description: "",
        duration: 300,
        leftFreq: 440,
        rightFreq: 445,
        volume: 50,
        durationMinutes: 5,
      },
    ],
  }
}

// Initialize volume
onMounted(() => {
  updateVolume()
  syncManualValues() // Initialize manual values

  // Load dark mode preference
  const savedDarkMode = localStorage.getItem("binaural-dark-mode")
  if (savedDarkMode !== null) {
    isDarkMode.value = savedDarkMode === "true"
  }
})

// Watch frequency mode changes
watch(frequencyMode, () => {
  handleFrequencyModeChange()
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
  color: #2c3e50;
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
  color: #2c3e50 !important;
}

/* Frequency Mode Toggle */
.frequency-mode-toggle {
  margin-bottom: 20px;
}

.toggle-buttons {
  display: flex;
  gap: 0;
  border-radius: 6px;
  overflow: hidden;
  border: 1px solid #ced4da;
}

.mode-btn {
  flex: 1;
  padding: 10px 16px;
  background: #f8f9fa;
  color: #495057;
  border: none;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s;
  border-right: 1px solid #ced4da;
}

.mode-btn:last-child {
  border-right: none;
}

.mode-btn:hover {
  background: #e9ecef;
  color: #2c3e50;
}

.mode-btn.active {
  background: #007bff;
  color: white;
}

.frequency-hint {
  font-size: 12px;
  color: #6c757d;
  font-weight: normal;
  display: block;
  margin-top: 2px;
}

.frequency-warning {
  font-size: 12px;
  color: #856404;
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  border-radius: 4px;
  padding: 4px 8px;
  margin-top: 5px;
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
  color: #495057;
  background: white;
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
  color: #495057;
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

/* Session Controls Styles */
.session-controls {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e9ecef;
  color: #242424;
  margin-bottom: 20px;
}

.preset-selection {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  color: #242424;
}

.preset-selection select {
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  background: white;
  min-width: 200px;
  color: #242424;
}

.edit-preset-btn {
  padding: 8px 16px;
  background: #17a2b8;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.edit-preset-btn:hover:not(:disabled) {
  background: #138496;
}

.edit-preset-btn:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.session-timer {
  margin-bottom: 20px;
}

.timer-controls {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}

.timer-controls select {
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  background: white;
  color: #242424;
}

.timer-display {
  background: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  color: #495057;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: #e9ecef;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 10px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #28a745, #20c997);
  transition: width 0.3s ease;
}

.time-remaining {
  text-align: center;
  font-weight: 600;
  color: #495057;
}

.session-info {
  background: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  color: #495057;
}

.session-info h4 {
  margin: 0 0 10px 0;
  color: #2c3e50;
}

.program-stage {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.stage-label {
  font-weight: 600;
  color: #495057;
}

.stage-description {
  color: #6c757d;
  font-style: italic;
}

.stage-progress {
  margin-top: 8px;
}

.stage-time {
  font-size: 14px;
  color: #007bff;
  font-weight: 600;
}

/* Session Plan Styles */
.session-plan {
  background: white;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid #dee2e6;
  color: #495057;
  margin-top: 15px;
}

.plan-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.plan-header h4 {
  margin: 0;
  color: #2c3e50;
}

.toggle-plan-btn {
  background: #f8f9fa;
  border: 1px solid #ced4da;
  border-radius: 4px;
  padding: 6px 12px;
  cursor: pointer;
  font-size: 14px;
  color: #495057;
  transition: all 0.2s;
}

.toggle-plan-btn:hover {
  background: #e9ecef;
}

.toggle-plan-btn.expanded {
  background: #007bff;
  color: white;
  border-color: #007bff;
}

.plan-content {
  margin-top: 15px;
}

.session-overview {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 4px;
}

.overview-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.overview-label {
  font-size: 12px;
  color: #6c757d;
  margin-bottom: 4px;
  font-weight: 600;
}

.overview-value {
  font-size: 16px;
  color: #2c3e50;
  font-weight: 600;
}

.progress-map {
  margin-bottom: 25px;
}

.map-title {
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 10px;
}

.timeline {
  display: flex;
  height: 60px;
  border-radius: 4px;
  overflow: hidden;
  border: 1px solid #dee2e6;
}

.timeline-segment {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #f8f9fa;
  border-right: 1px solid #dee2e6;
  position: relative;
  padding: 5px;
  transition: all 0.3s;
}

.timeline-segment:last-child {
  border-right: none;
}

.timeline-segment.completed {
  background: #d4edda;
  border-color: #c3e6cb;
}

.timeline-segment.active {
  background: #cce7ff;
  border-color: #007bff;
  animation: pulse 2s infinite;
}

.timeline-segment.current {
  background: #fff3cd;
  border-color: #ffc107;
}

@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.7;
  }
  100% {
    opacity: 1;
  }
}

.segment-label {
  font-size: 11px;
  font-weight: 600;
  color: #495057;
  text-align: center;
  line-height: 1.2;
  margin-bottom: 2px;
}

.segment-time {
  font-size: 10px;
  color: #6c757d;
}

.stages-detail {
  margin-top: 20px;
}

.detail-title {
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 15px;
  font-size: 16px;
}

.stage-detail {
  border: 1px solid #dee2e6;
  border-radius: 4px;
  margin-bottom: 10px;
  background: #fafafa;
  transition: all 0.3s;
}

.stage-detail.completed {
  background: #d4edda;
  border-color: #c3e6cb;
}

.stage-detail.active {
  background: #cce7ff;
  border-color: #007bff;
  box-shadow: 0 2px 4px rgba(0, 123, 255, 0.1);
}

.stage-header-detail {
  display: flex;
  align-items: center;
  padding: 12px 15px;
  border-bottom: 1px solid #dee2e6;
}

.stage-number {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #e9ecef;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: #495057;
  margin-right: 12px;
  font-size: 14px;
}

.stage-detail.completed .stage-number {
  background: #28a745;
  color: white;
}

.stage-detail.active .stage-number {
  background: #007bff;
  color: white;
}

.stage-info {
  flex: 1;
}

.stage-name {
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 2px;
}

.stage-duration {
  font-size: 14px;
  color: #6c757d;
}

.stage-status {
  margin-left: 10px;
}

.status-completed {
  color: #28a745;
  font-size: 18px;
  font-weight: 600;
}

.status-active {
  color: #007bff;
  font-size: 16px;
}

.status-pending {
  color: #6c757d;
  font-size: 16px;
}

.stage-frequencies {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 10px;
  padding: 12px 15px;
  background: rgba(255, 255, 255, 0.5);
}

.freq-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.freq-label {
  font-size: 11px;
  color: #6c757d;
  margin-bottom: 2px;
  font-weight: 600;
}

.freq-value {
  font-size: 13px;
  color: #2c3e50;
  font-weight: 600;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.preset-editor-modal {
  background: white;
  border-radius: 8px;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #dee2e6;
}

.modal-header h3 {
  margin: 0;
  color: #2c3e50;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #6c757d;
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-btn:hover {
  color: #495057;
}

.modal-content {
  padding: 20px;
}

.preset-list {
  margin-bottom: 30px;
}

.preset-list h4,
.create-preset h4 {
  color: #2c3e50;
  margin-bottom: 15px;
}

.preset-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 4px;
  margin-bottom: 10px;
}

.preset-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.preset-info strong {
  color: #2c3e50;
}

.preset-description {
  color: #6c757d;
  font-size: 14px;
}

.preset-duration {
  color: #007bff;
  font-weight: 600;
  font-size: 14px;
}

.preset-actions {
  display: flex;
  gap: 8px;
}

.edit-btn,
.delete-btn {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.edit-btn {
  background: #ffc107;
  color: #212529;
}

.edit-btn:hover {
  background: #e0a800;
}

.delete-btn {
  background: #dc3545;
  color: white;
}

.delete-btn:hover {
  background: #c82333;
}

.preset-form {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 4px;
  color: #495057;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
  margin-bottom: 15px;
}

.form-group label {
  font-weight: 600;
  color: #495057;
}

.form-group input {
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  color: #495057;
  background: white;
}

.stages-section {
  margin-top: 20px;
}

.stages-section h5 {
  color: #2c3e50;
  margin-bottom: 15px;
}

.stage-editor {
  background: white;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  padding: 15px;
  margin-bottom: 15px;
  color: #495057;
}

.stage-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.stage-header h6 {
  margin: 0;
  color: #2c3e50;
}

.remove-stage-btn {
  background: #dc3545;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.remove-stage-btn:hover {
  background: #c82333;
}

.stage-controls {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.control-group label {
  font-weight: 600;
  color: #495057;
  font-size: 14px;
}

.control-group input {
  padding: 6px 10px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 14px;
  color: #495057;
  background: white;
}

.add-stage-btn {
  background: #28a745;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.add-stage-btn:hover {
  background: #218838;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
  justify-content: flex-end;
}

.save-btn,
.cancel-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;
}

.save-btn {
  background: #007bff;
  color: white;
}

.save-btn:hover {
  background: #0056b3;
}

.cancel-btn {
  background: #6c757d;
  color: white;
}

.cancel-btn:hover {
  background: #545b62;
}

@media (max-width: 768px) {
  .preset-selection {
    flex-direction: column;
    align-items: stretch;
  }

  .preset-selection select {
    min-width: auto;
  }

  .timer-controls {
    flex-direction: column;
    align-items: stretch;
  }

  .stage-controls {
    grid-template-columns: 1fr;
  }

  .preset-item {
    flex-direction: column;
    align-items: stretch;
    gap: 10px;
  }

  .preset-actions {
    justify-content: center;
  }

  .form-actions {
    flex-direction: column;
  }
}

/* Dark Mode Styles */
.dark-mode {
  background: #1a1a1a;
  color: #e0e0e0;
}

.dark-mode .app-header h1 {
  color: #ffffff;
}

.dark-mode .app-header p {
  color: #b0b0b0;
}

.dark-mode .theme-toggle {
  background: #333333;
  color: #e0e0e0;
  border: 1px solid #555555;
}

.dark-mode .theme-toggle:hover {
  background: #444444;
  color: #ffffff;
}

.dark-mode .frequency-controls,
.dark-mode .player-controls {
  background: #2a2a2a;
  border: 1px solid #444444;
  color: #e0e0e0;
}

.dark-mode .frequency-controls h2,
.dark-mode .player-controls h2,
.dark-mode .noise-controls h3 {
  color: #ffffff !important;
}

.dark-mode .mode-btn {
  background: #333333;
  color: #e0e0e0;
  border-color: #555555;
}

.dark-mode .mode-btn:hover {
  background: #444444;
  color: #ffffff;
}

.dark-mode .mode-btn.active {
  background: rgb(86, 154, 211);
  color: white;
}

.dark-mode .frequency-input label {
  color: #e0e0e0;
}

.dark-mode .frequency-input input[type="number"] {
  background: #333333;
  border: 1px solid #555555;
  color: #e0e0e0;
}

.dark-mode .frequency-input input[type="number"]:focus {
  border-color: rgb(86, 154, 211);
}

.dark-mode .hz-label {
  color: #b0b0b0;
}

.dark-mode .frequency-data {
  background: #333333;
  border: 1px solid #555555;
  color: #e0e0e0;
}

.dark-mode .label {
  color: #e0e0e0;
}

.dark-mode .frequency-warning {
  background: #3d3d1a;
  border-color: #5a5a2a;
  color: #cccc66;
}

.dark-mode .frequency-hint {
  color: #b0b0b0;
}

.dark-mode .player-controls button {
  background: rgb(86, 154, 211);
  color: white;
  border: 1px solid rgb(86, 154, 211);
}

.dark-mode .player-controls button:hover:not(:disabled) {
  background: rgba(86, 154, 211, 0.8);
}

.dark-mode .player-controls button:disabled {
  background: #444444;
  color: #888888;
  border-color: #444444;
}

.dark-mode .volume-control input[type="range"],
.dark-mode .session-duration input[type="range"] {
  background: #333333;
}

.dark-mode .volume-control input[type="range"]::-webkit-slider-track,
.dark-mode .session-duration input[type="range"]::-webkit-slider-track {
  background: #555555;
}

.dark-mode .volume-control input[type="range"]::-webkit-slider-thumb,
.dark-mode .session-duration input[type="range"]::-webkit-slider-thumb {
  background: rgb(86, 154, 211);
}

.dark-mode .volume-control select,
.dark-mode .session-duration select {
  background: #333333;
  color: #e0e0e0;
  border: 1px solid #555555;
}

.dark-mode .volume-control select:focus,
.dark-mode .session-duration select:focus {
  border-color: rgb(86, 154, 211);
}

.dark-mode .volume-control label,
.dark-mode .session-duration label,
.dark-mode .noise-controls label {
}

.dark-mode .noise-controls input[type="range"] {
  background: #333333;
}

.dark-mode .noise-controls input[type="range"]::-webkit-slider-track {
  background: #555555;
}

.dark-mode .noise-controls input[type="range"]::-webkit-slider-thumb {
  background: rgb(86, 154, 211);
}

.dark-mode .preset-card {
  background: #2a2a2a;
  border: 1px solid #444444;
}

.dark-mode .preset-card:hover {
  border-color: rgb(86, 154, 211);
}

.dark-mode .preset-card h3 {
  color: #ffffff;
}

.dark-mode .preset-card p {
  color: #b0b0b0;
}

.dark-mode .preset-card .preset-frequency {
  color: rgb(86, 154, 211);
}

.dark-mode .load-preset-btn {
  background: rgb(86, 154, 211);
  color: white;
  border: 1px solid rgb(86, 154, 211);
}

.dark-mode .load-preset-btn:hover {
  background: rgba(86, 154, 211, 0.8);
}

.dark-mode .session-controls {
  background: #2a2a2a;
  border: 1px solid #444444;
}

.dark-mode .session-controls h2 {
  color: #ffffff !important;
}

.dark-mode .session-controls label {
  color: #e0e0e0;
}

.dark-mode .custom-presets h2 {
  color: #ffffff !important;
}

.dark-mode .create-preset-btn {
  background: rgb(86, 154, 211);
  color: white;
  border: 1px solid rgb(86, 154, 211);
}

.dark-mode .create-preset-btn:hover {
  background: rgba(86, 154, 211, 0.8);
}

.dark-mode .preset-form {
  background: #2a2a2a;
  border: 1px solid #444444;
}

.dark-mode .preset-form input[type="text"],
.dark-mode .preset-form textarea,
.dark-mode .preset-form input[type="number"] {
  background: #333333;
  border: 1px solid #555555;
  color: #e0e0e0;
}

.dark-mode .preset-form input[type="text"]:focus,
.dark-mode .preset-form textarea:focus,
.dark-mode .preset-form input[type="number"]:focus {
  border-color: rgb(86, 154, 211);
}

.dark-mode .preset-form label {
  color: #e0e0e0;
}

.dark-mode .form-actions button {
  background: rgb(86, 154, 211);
  color: white;
  border: 1px solid rgb(86, 154, 211);
}

.dark-mode .form-actions button:hover {
  background: rgba(86, 154, 211, 0.8);
}

.dark-mode .form-actions button.cancel {
  background: #666666;
  color: white;
  border: 1px solid #666666;
}

.dark-mode .form-actions button.cancel:hover {
  background: #777777;
}

.dark-mode .delete-preset-btn {
  background: #dc3545;
  color: white;
  border: 1px solid #dc3545;
}

.dark-mode .delete-preset-btn:hover {
  background: #c82333;
}
</style>
