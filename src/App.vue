<script setup>
import GameBoard from "@/components/GameBoard.vue"
import Solver from "@/components/Solver.vue"
</script>

<template>
  <h1 style="text-align: center; font-size: 2.5rem; font-weight: bold; letter-spacing: 0.2rem; margin: 20px;">
    WORDLE SOLVER
  </h1>

  <main>
    <GameBoard />
    <Solver />
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: row;
  gap: 200px;
}
</style>
