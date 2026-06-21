<script setup>
import { ref, computed, onMounted } from 'vue'
import { store } from '@/store.js'

const allLetters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('')

// Pagination variables
const words = ref([])
const page = ref(1)
const pageSize = 60

const totalPages = computed(() =>
    Math.ceil(filteredWords.value.length / pageSize)
)

const paginatedWords = computed(() => {
  const start = (page.value - 1) * pageSize
  return filteredWords.value.slice(start, start + pageSize)
})

// Filtering
const filteredWords = computed(() => {
  return getPossibleWords()
})

function getPossibleWords() {
  let possibleLetters = Array.from({ length: 5 }, () => [...allLetters])

  // Gray letters
  store.grayLetters.forEach(letter => {
    for (let i = 0; i < 5; i++) {
      possibleLetters[i] = possibleLetters[i].filter(l => l !== letter)
    }
  })

  // Yellow letters
  store.yellowLetters.forEach(({ letter, slot }) => {
    possibleLetters[slot] = possibleLetters[slot].filter(l => l !== letter)
  })

  // Green letters
  store.greenLetters.forEach(({ letter, slot }) => {
    possibleLetters[slot] = [letter]
  })

  // Update possible words
  return words.value.filter(word =>
      // Remove words that contain at least one impossible letter
      [...word.toUpperCase()].every((letter, index) =>
          possibleLetters[index].includes(letter)
      ) &&
      // Remove words that don't contain any yellow letters
      store.yellowLetters.every(({ letter }) => word.toUpperCase().includes(letter))
  )
}

// Pagination methods
function nextPage() {
  if (page.value < totalPages.value) {
    page.value++
  } else {
    page.value = 1
  }
}

function previousPage() {
  if (page.value > 1) {
    page.value--
  } else {
    page.value = totalPages.value
  }
}

// Get words from words.txt
onMounted(async () => {
  const response = await fetch('/words.txt')
  words.value = (await response.text()).split(/\r?\n/)
})
</script>

<template>
  <div class="solver">
    <div class="container">
      <h2 style="text-align: center; letter-spacing: 0.08rem">
        Possible Words ({{ filteredWords.length }})
      </h2>

      <div class="words">
        <div
            v-for="word in paginatedWords"
            :key="word"
            class="word"
        >
          {{ word }}
        </div>
      </div>
    </div>

    <div class="pagination">
      <button class="page-button" @click="page=1">
        <<
      </button>

      <button class="page-button" @click="previousPage">
        <
      </button>

      <h3 class="page-number">
        {{ page }} / {{ totalPages }}
      </h3>

      <button class="page-button" @click="nextPage">
        >
      </button>

      <button class="page-button" @click="page=totalPages">
        >>
      </button>
    </div>
  </div>
</template>

<style scoped>
.solver {
  width: 400px;

  display: flex;
  flex-direction: column;
  align-items: center;
}

.container {
  height: 515px;
}

.words {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
}

.word {
  width: 70px;
  height: 30px;

  display: flex;
  justify-content: center;
  align-items: center;

  background-color: #3a3a3c;
  border-radius: 4px;

  text-transform: uppercase;
  letter-spacing: 0.08rem;

  margin: 5px;
}

.pagination {
  display: flex;
  flex-direction: row;

  justify-content: center;
  align-items: center;
}

.page-number {
  display: flex;
  justify-content: center;
  align-items: center;

  width: 110px;
  height: 35px;

  background-color: #1f1f20;

  margin: 0 5px;
  padding: 0 5px;
  border-radius: 4px;
}

.page-button {
  background-color: #818384;
  border: none;

  color: white;
  font-size: 1.2rem;

  padding: 5px 20px;
  margin: 5px;
  border-radius: 4px;

  width: 60px;

  text-align: center;
  cursor: pointer;
}
</style>