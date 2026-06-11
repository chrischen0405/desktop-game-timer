<script setup lang="ts">
import { ref, computed } from 'vue'

interface Player {
  id: number
  name: string
  time: number
  sessionTime: number
  isActive: boolean
}

const players = ref<Player[]>([
  { id: 1, name: '玩家A', time: 0, sessionTime: 0, isActive: false },
  { id: 2, name: '玩家B', time: 0, sessionTime: 0, isActive: false },
  { id: 3, name: '玩家C', time: 0, sessionTime: 0, isActive: false },
  { id: 4, name: '玩家D', time: 0, sessionTime: 0, isActive: false }
])

const currentPlayerId = ref<number | null>(null)
let timerInterval: number | null = null
const showSettings = ref(false)
const isPaused = ref(false)
const showResetConfirm = ref(false)
const pausedPlayerId = ref<number | null>(null)

const formatTime = (seconds: number): string => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

const activePlayer = computed(() => {
  return players.value.find(p => p.isActive)
})

const totalTime = computed(() => {
  return players.value.reduce((sum, p) => sum + p.time, 0)
})

const startTimer = (playerId: number) => {
  // 如果点击的是当前正在计时的玩家，暂停该玩家并开始下一位
  if (currentPlayerId.value === playerId) {
    if (timerInterval) {
      clearInterval(timerInterval)
      timerInterval = null
    }
    isPaused.value = false

    // 找到当前玩家的下一个玩家
    const currentIndex = players.value.findIndex(p => p.id === playerId)
    const nextIndex = (currentIndex + 1) % players.value.length
    const nextPlayer = players.value[nextIndex]

    // 重置下一个玩家的单次计时
    nextPlayer.sessionTime = 0

    players.value.forEach(p => {
      p.isActive = p.id === nextPlayer.id
    })
    currentPlayerId.value = nextPlayer.id

    timerInterval = window.setInterval(() => {
      const player = players.value.find(p => p.id === nextPlayer.id)
      if (player) {
        player.time++
        player.sessionTime++
      }
    }, 1000)
    return
  }

  if (timerInterval) {
    clearInterval(timerInterval)
  }

  isPaused.value = false

  // 重置新玩家的单次计时
  const newPlayer = players.value.find(p => p.id === playerId)
  if (newPlayer) {
    newPlayer.sessionTime = 0
  }

  players.value.forEach(p => {
    p.isActive = p.id === playerId
  })
  currentPlayerId.value = playerId

  timerInterval = window.setInterval(() => {
    const player = players.value.find(p => p.id === playerId)
    if (player) {
      player.time++
      player.sessionTime++
    }
  }, 1000)
}

const pauseTimer = () => {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
    isPaused.value = true
    pausedPlayerId.value = currentPlayerId.value
    players.value.forEach(p => {
      p.isActive = false
    })
  }
}

const resumeTimer = () => {
  if (pausedPlayerId.value && !timerInterval) {
    isPaused.value = false
    const playerId = pausedPlayerId.value
    players.value.forEach(p => {
      p.isActive = p.id === playerId
    })
    currentPlayerId.value = playerId

    timerInterval = window.setInterval(() => {
      const player = players.value.find(p => p.id === playerId)
      if (player) {
        player.time++
      }
    }, 1000)
  }
}

const resetAll = () => {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
  players.value.forEach(p => {
    p.time = 0
    p.sessionTime = 0
    p.isActive = false
  })
  currentPlayerId.value = null
  isPaused.value = false
  pausedPlayerId.value = null
  showResetConfirm.value = false
}

const showResetModal = () => {
  showResetConfirm.value = true
}

const cancelReset = () => {
  showResetConfirm.value = false
}

const updatePlayerName = (id: number, name: string) => {
  const player = players.value.find(p => p.id === id)
  if (player) {
    player.name = name || `玩家${String.fromCharCode(64 + id)}`
  }
}

const addPlayer = () => {
  const newId = players.value.length + 1
  players.value.push({
    id: newId,
    name: `玩家${String.fromCharCode(64 + newId)}`,
    time: 0,
    sessionTime: 0,
    isActive: false
  })
}

const removePlayer = (id: number) => {
  if (players.value.length <= 1) return
  const index = players.value.findIndex(p => p.id === id)
  if (index > -1) {
    if (players.value[index].isActive && timerInterval) {
      clearInterval(timerInterval)
      timerInterval = null
      currentPlayerId.value = null
    }
    players.value.splice(index, 1)
    players.value.forEach((p, i) => {
      p.id = i + 1
    })
  }
}

const toggleSettings = () => {
  showSettings.value = !showSettings.value
}
</script>

<template>
  <div class="game-timer">
    <div class="settings-btn" @click="toggleSettings">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2">
        <circle cx="12" cy="12" r="3"></circle>
        <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09a1.65 1.65 0 0 0 1.51-1 1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path>
      </svg>
    </div>

    <div v-if="showSettings" class="settings-panel">
      <div class="panel-header">
        <h3>玩家设置</h3>
        <button class="close-btn" @click="toggleSettings">×</button>
      </div>
      <div class="player-list">
        <div v-for="player in players" :key="player.id" class="player-item">
          <input
            type="text"
            :value="player.name"
            @input="updatePlayerName(player.id, ($event.target as HTMLInputElement).value)"
            class="name-input"
          />
          <button
            v-if="players.length > 1"
            class="remove-btn"
            @click="removePlayer(player.id)"
          >
            ×
          </button>
        </div>
      </div>
      <button class="add-player-btn" @click="addPlayer">
        + 添加玩家
      </button>
    </div>

    <div class="timer-container">
      <div class="top-panel">
        <div class="total-time">
          <span class="label">总用时</span>
          <span class="time">{{ formatTime(totalTime) }}</span>
        </div>
        <div v-if="isPaused" class="active-indicator paused">
          <span class="active-text">⏸️ 已暂停</span>
          <button class="resume-btn" @click="resumeTimer">继续</button>
        </div>
        <div v-else-if="activePlayer" class="active-indicator">
          <span class="active-text">{{ activePlayer.name }} 进行中</span>
        </div>
        <div class="top-buttons">
          <button v-if="!isPaused && activePlayer" class="pause-btn" @click="pauseTimer">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2">
              <rect x="6" y="4" width="4" height="16"></rect>
              <rect x="14" y="4" width="4" height="16"></rect>
            </svg>
            暂停
          </button>
          <button class="reset-btn" @click="showResetModal">重置</button>
        </div>
      </div>

      <div class="player-sections">
        <div
          v-for="player in players"
          :key="player.id"
          :class="['player-section', { active: player.isActive }]"
          @click="startTimer(player.id)"
        >
          <div class="player-name">{{ player.name }}</div>
          <div class="player-time">{{ formatTime(player.time) }}</div>
          <div class="player-session">
            <span class="session-label">本轮</span>
            <span class="session-time">{{ formatTime(player.sessionTime) }}</span>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showResetConfirm" class="modal-overlay" @click.self="cancelReset">
      <div class="modal-content">
        <div class="modal-header">
          <h3>确认重置</h3>
        </div>
        <div class="modal-body">
          <p>确定要重置所有玩家的计时数据吗？此操作无法撤销。</p>
        </div>
        <div class="modal-footer">
          <button class="cancel-btn" @click="cancelReset">取消</button>
          <button class="confirm-btn" @click="resetAll">确认重置</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-timer {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  padding: 20px;
  position: relative;
}

.settings-btn {
  position: fixed;
  top: 20px;
  right: 20px;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  border: 1px solid rgba(255, 255, 255, 0.2);
  z-index: 100;
  transition: background 0.3s;

  &:hover {
    background: rgba(255, 255, 255, 0.2);
  }
}

.settings-panel {
  position: fixed;
  top: 80px;
  right: 20px;
  width: 280px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border-radius: 16px;
  padding: 20px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  z-index: 99;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;

  h3 {
    color: white;
    font-size: 18px;
    margin: 0;
  }
}

.close-btn {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: white;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;

  &:hover {
    background: rgba(255, 255, 255, 0.2);
  }
}

.player-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.player-item {
  display: flex;
  gap: 10px;
}

.name-input {
  flex: 1;
  padding: 10px 14px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 14px;
  outline: none;

  &:focus {
    border-color: #4a69bd;
  }

  &::placeholder {
    color: rgba(255, 255, 255, 0.5);
  }
}

.remove-btn {
  width: 32px;
  height: 32px;
  border-radius: 8px;
  background: rgba(239, 83, 80, 0.2);
  border: 1px solid rgba(239, 83, 80, 0.4);
  color: #ef5350;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;

  &:hover {
    background: rgba(239, 83, 80, 0.3);
  }
}

.add-player-btn {
  width: 100%;
  margin-top: 16px;
  padding: 12px;
  border: 2px dashed rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  background: transparent;
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;

  &:hover {
    border-color: rgba(255, 255, 255, 0.5);
    background: rgba(255, 255, 255, 0.05);
  }
}

.timer-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 600px;
}

.top-panel {
  width: 100%;
  padding: 24px;
  border-radius: 20px;
  background: linear-gradient(135deg, #4a69bd 0%, #6a89cc 50%, #82ccdd 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 0 10px 40px rgba(74, 105, 189, 0.4),
              inset 0 2px 10px rgba(255, 255, 255, 0.3);
  margin-bottom: 40px;
}

.total-time {
  text-align: center;
}

.label {
  display: block;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 8px;
}

.time {
  font-size: 48px;
  font-weight: bold;
  color: white;
  font-family: 'Courier New', monospace;
}

.active-indicator {
  margin-top: 16px;

  &.paused {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
  }
}

.active-text {
  font-size: 16px;
  color: rgba(255, 255, 255, 0.9);
  background: rgba(0, 0, 0, 0.2);
  padding: 6px 16px;
  border-radius: 20px;
}

.resume-btn {
  padding: 8px 20px;
  border: none;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.3s;

  &:hover {
    background: rgba(255, 255, 255, 0.3);
  }
}

.top-buttons {
  display: flex;
  gap: 16px;
  margin-top: 20px;
}

.pause-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border: none;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-size: 15px;
  cursor: pointer;
  transition: background 0.3s;

  &:hover {
    background: rgba(255, 255, 255, 0.3);
  }
}

.reset-btn {
  padding: 10px 24px;
  border: none;
  border-radius: 20px;
  background: rgba(239, 83, 80, 0.3);
  color: white;
  font-size: 15px;
  cursor: pointer;
  transition: background 0.3s;

  &:hover {
    background: rgba(239, 83, 80, 0.4);
  }
}

.player-sections {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  width: 100%;
}

.player-section {
  padding: 24px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  border: 2px solid rgba(255, 255, 255, 0.1);
  min-height: 100px;
  transition: all 0.3s ease;

  &:hover {
    background: rgba(255, 255, 255, 0.15);
    border-color: rgba(255, 255, 255, 0.3);
    transform: translateY(-2px);
  }

  &.active {
    background: linear-gradient(135deg, #26de81 0%, #20bf6b 100%);
    border-color: #26de81;
    box-shadow: 0 10px 30px rgba(38, 222, 129, 0.4);
    transform: translateY(-2px);
  }
}

.player-name {
  font-size: 20px;
  font-weight: bold;
  color: white;
  margin-bottom: 8px;
}

.player-time {
  font-size: 28px;
  font-weight: bold;
  color: white;
  font-family: 'Courier New', monospace;
}

.player-session {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 12px;
  padding: 6px 12px;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 12px;
}

.session-label {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.7);
}

.session-time {
  font-size: 16px;
  font-weight: bold;
  color: white;
  font-family: 'Courier New', monospace;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(5px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 200;
}

.modal-content {
  width: 320px;
  background: rgba(30, 41, 59, 0.95);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  overflow: hidden;
}

.modal-header {
  padding: 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);

  h3 {
    margin: 0;
    color: white;
    font-size: 18px;
    text-align: center;
  }
}

.modal-body {
  padding: 24px 20px;

  p {
    margin: 0;
    color: rgba(255, 255, 255, 0.8);
    font-size: 14px;
    text-align: center;
    line-height: 1.5;
  }
}

.modal-footer {
  display: flex;
  gap: 10px;
  padding: 16px 20px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.cancel-btn {
  flex: 1;
  padding: 12px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  background: transparent;
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;

  &:hover {
    background: rgba(255, 255, 255, 0.1);
  }
}

.confirm-btn {
  flex: 1;
  padding: 12px;
  border: none;
  border-radius: 8px;
  background: rgba(239, 83, 80, 0.8);
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.3s;

  &:hover {
    background: rgba(239, 83, 80, 1);
  }
}
</style>
