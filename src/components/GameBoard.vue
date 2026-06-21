<script setup>
import { ref } from 'vue'
import { onMounted, onUnmounted } from 'vue';
import { store } from '@/store.js'

// Color constants
const WHITE = '#818384'
const GRAY = '#3a3a3c'
const YELLOW = '#b59f3b'
const GREEN = '#538d4e'

// Grid data
const board = ref(
    Array.from({ length: 5 }, () =>
        Array.from({ length: 5 }, () => ({
          letter: '',
          color: ''
        }))
    )
)

// Keyboard data
const keyboard = [
  ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
  ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'],
  ['ENTER', 'Z', 'X', 'C', 'V', 'B', 'N', 'M', 'BACKSPACE']
]

const keyColors = ref(
    Object.fromEntries(
        'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
            .split('')
            .map(letter => [letter, WHITE])
    )
)

// Grid methods
function setTileLetter(row, col, letter) {
  board.value[row][col].letter = letter
}

function getTileLetter(row, col) {
  return board.value[row][col].letter
}

function setTileColor(row, col, color) {
  board.value[row][col].color = color
}

function getTileColor(row, col) {
  return board.value[row][col].color
}

function cycleTileColor(row, col) {
  if (getTileLetter(row, col) === '') { return }

  const tileColor = getTileColor(row, col)
  const tileLetter = getTileLetter(row, col)

  if (tileColor === GRAY) {
    setTileColor(row, col, YELLOW)
  } else if (tileColor === YELLOW) {
    // Prevent conflicting green slots
    for (let i = 0; i < 6; i++) {
      if (getTileColor(i, col) === GREEN && getTileLetter(i, col) !== tileLetter) {
        setTileColor(row, col, GRAY)
        break
      } else {
        setTileColor(row, col, GREEN)
      }
    }
  } else {
    setTileColor(row, col, GRAY)
  }
}

function getPos() {
  for (let i = 0; i < 5; i++) {
    for (let j = 0; j < 5; j++) {
      if (getTileLetter(i, j) === '') {
        return [i, j]
      }
    }
  }
  return [5, 0]
}

// Keyboard methods
function setKeyColor(key, color) {
  keyColors.value[key] = color
}

function getKeyColor(key) {
  return keyColors.value[key]
}

function addLetter(letter) {
  const pos = getPos()
  if (pos[0] === 6 && pos[1] === 0) { return }

  setTileLetter(pos[0], pos[1], letter)
  setTileColor(pos[0], pos[1], GRAY)
}

function backspace() {
  const pos = getPos()
  let prevPos = pos

  if (pos[0] === 0 && pos[1] === 0) {
    return
  } else if (pos[1] === 0) {
    prevPos = [pos[0]-1, 4]
  } else {
    prevPos = [pos[0], pos[1]-1]
  }

  setTileLetter(prevPos[0], prevPos[1], '')
  setTileColor(prevPos[0], prevPos[1], '')
}

function colorPriority(color) {
  if (color === GREEN) return 3
  if (color === YELLOW) return 2
  if (color === GRAY) return 1
  return 0
}

function updateColorsAndLetters() {
  const grayLetters = []
  const yellowLetters = []
  const greenLetters = []

  // Reset all key colors
  for (const letter of 'ABCDEFGHIJKLMNOPQRSTUVWXYZ') {
    setKeyColor(letter, WHITE)
  }

  // Re-apply colors only for non-empty tiles, with priority
  for (let i = 0; i < 6; i++) {
    for (let j = 0; j < 5; j++) {
      const letter = getTileLetter(i, j)
      const color = getTileColor(i, j)

      if (!letter) continue

      if (color === GRAY) grayLetters.push(letter)
      else if (color === YELLOW) yellowLetters.push({letter: letter, slot: j})
      else if (color === GREEN) greenLetters.push({letter: letter, slot: j})

      if (colorPriority(color) > colorPriority(getKeyColor(letter))) {
        setKeyColor(letter, color || WHITE)
      }
    }
  }

  store.grayLetters = grayLetters
  store.yellowLetters = yellowLetters
  store.greenLetters = greenLetters
}

function pressKey(key) {
  key = key.toUpperCase()
  if (key === 'ENTER') {
    updateColorsAndLetters()
  } else if (key === 'BACKSPACE') {
    backspace()
  } else if (/^[A-Z]$/.test(key)) {
    addLetter(key)
  }
}

// Physical keyboard handling
function handleKeydown(event) {
  event.target.blur()
  pressKey(event.key)
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
})
</script>

<template>
  <div class="game-board">
    <!-- Grid Section -->
    <div class="grid">
      <div
          v-for="(row, rowIndex) in board"
          :key="rowIndex"
          class="row"
      >
        <button
            v-for="(tile, colIndex) in row"
            :key="colIndex"
            class="tile"
            :style="{ backgroundColor: tile.color, borderColor: tile.color }"
            @click="cycleTileColor(rowIndex, colIndex)"
        >
          {{ tile.letter }}
        </button>
      </div>

      <div class="row">
        <div
            v-for="(tile, colIndex) in 5"
            :key="colIndex"
            class="tile"
            style="border-color: #1e1e21"
        >
          {{ tile.letter }}
        </div>
      </div>
    </div>

    <!-- Keyboard Section -->
    <div class="keyboard">
      <div
          v-for="(row, rowIndex) in keyboard"
          :key="rowIndex"
          class="row"
      >
        <button
            v-for="key in row"
            :key="key"
            :class="[
                'key',
                { special: key === 'ENTER' || key === 'BACKSPACE'}
            ]"
            :style="{ backgroundColor: getKeyColor(key) }"
            @click="pressKey(key)"
        >
          <template v-if="key !== 'BACKSPACE'">
            {{ key }}
          </template>

          <template v-else>
            <svg
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 24 24"
                fill="currentColor"
                width="20px"
                height="20px"
            >
              <path d="M22 3H7c-.69 0-1.23.35-1.59.88L0 12l5.41 8.11c.36.53.9.89 1.59.89h15c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm0 16H7.07L2.4 12l4.66-7H22v14zm-11.59-2L14 13.41 17.59 17 19 15.59 15.41 12 19 8.41 17.59 7 14 10.59 10.41 7 9 8.41 12.59 12 9 15.59z"/>
            </svg>
          </template>

        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-board {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

/* Grid Styles */
.grid {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
}

.grid .row {
  display: flex;
  flex-direction: row;
  gap: 6px;
}

.tile {
  width: 50px;
  height: 50px;

  background-color: transparent;
  border-color: #3a3a3c;
  border-style: solid;
  border-width: 2px;

  display: flex;
  justify-content: center;
  align-items: center;

  font-size: 1.8rem;
  font-weight: bold;
  color: #f8f8f8;
  text-transform: uppercase;
}

/* Keyboard Styles */
.keyboard {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 9px;
  margin-top: 10px;
}

.keyboard .row {
  display: flex;
  flex-direction: row;
  gap: 6px;
}

.key {
  width: 45px;
  height: 60px;

  background-color: #818384;
  border: none;
  border-radius: 4px;

  display: flex;
  justify-content: center;
  align-items: center;

  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  font-weight: bold;
  font-size: 1.5rem;
  color: #ffffff;
  text-transform: uppercase;

  cursor: pointer;
}

.key.special {
  width: 70px;
  font-size: 0.8rem;
}
</style>
