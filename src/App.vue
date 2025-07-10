<template>
  <div class="container-1 flex flex-col items-center justify-center min-h-screen bg-gray-100 px-y-5">
    <h1 class="text-3xl font-semibold pb-5 text-gray-50 text-shadow-amber-50">🧠Trivia Quiz Game</h1>


    <div v-if="loading" class="flex items-center justify-center ">
      <div
        class="px-3 py-1 text-base font-medium leading-none text-center text-blue-800 bg-blue-200 rounded-full animate-pulse dark:bg-blue-900 dark:text-blue-200">
        loading question please wait...</div>
    </div>

    <div v-else-if="currentQuestion" class="mt-5 px-5 py-5">
      <div class=" pb-5 px-5 py-5 bg-white shadow-lg rounded-lg w-full max-w-2xl">
        <div class="flex justify-between items-center mb-4 text-sm">
          <p class="text-gray-500 font-semibold mb-4">{{ currentIndex + 1 }} of {{ total }}</p>
          <p class="text-gray-500 mb-4">{{ currentQuestion.category }}</p>
        </div>
        <p class="text-3xl text-gray-900 font-bold" v-html="decode(currentQuestion.question)"></p>
      </div>

      <div class="grid gap-2 pt-5">
        <button v-for="choice in shuffledChoices" :key="choice" @click="handleAnswer(choice)" :disabled="showAnswer"
          :class="[
            'text-white font-semibold bg-gradient-to-r from-cyan-500 to-blue-500 hover:bg-gradient-to-bl focus:ring-4 focus:outline-none focus:ring-cyan-300 dark:focus:ring-cyan-800 rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2',
            showAnswer
              ? choice === currentQuestion.correct_answer
                ? 'text-white bg-gradient-to-r from-green-400 via-green-500 to-green-600 hover:bg-gradient-to-br focus:ring-4 focus:outline-none focus:ring-green-300 dark:focus:ring-green-800 shadow-lg shadow-green-500/50 dark:shadow-lg dark:shadow-green-800/80 font-medium rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2'
                : 'text-white bg-gradient-to-r from-red-400 via-red-500 to-red-600 hover:bg-gradient-to-br focus:ring-4 focus:outline-none focus:ring-red-300 dark:focus:ring-red-800 shadow-lg shadow-red-500/50 dark:shadow-lg dark:shadow-red-800/80 font-medium rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2'
              : 'bg-white hover:bg-gray-100 border-gray-300'
          ]" v-html="decode(choice)" />
      </div>

      <div v-if="showAnswer" class="mt-5 mb-4 items-center pt-5">

        <div v-if="isCorrect" class="bg-green-200 px-6 py-4 my-4 rounded-md text-lg flex items-center mx-auto mt-5">
          <span class="text-green-800 ml-5">Your answer is CORRECT.</span>
        </div>

        <div v-else class="bg-red-200 px-6 py-4 my-4 rounded-md text-lg flex items-center mx-auto mt-5">
          <span class="text-red-800 ml-5"> Wrong answer! Correct answer is : <span
              v-html="decode(currentQuestion.correct_answer)"></span> </span>
        </div>

        <div class="flex justify-center space-x-4 pt-5">
          <button @click="nextQuestion"
            class="flex items-center text-white bg-gradient-to-r from-cyan-500 to-blue-500 hover:bg-gradient-to-bl focus:ring-4 focus:outline-none focus:ring-cyan-300 dark:focus:ring-cyan-800 font-medium rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2">
            Next Question
            <svg class="w-6 h-6 text-white dark:text-white" aria-hidden="true" xmlns="http://www.w3.org/2000/svg"
              width="24" height="24" fill="none" viewBox="0 0 24 24">
              <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M19 12H5m14 0-4 4m4-4-4-4" />
            </svg>

          </button>
        </div>

      </div>
    </div>

    <div v-if="gameOver" class="mt-5 text-center">
      <div class="px-5 py-5 shadow-lg rounded-lg w-full max-w-2xl">
        <h2 class="text-2xl font-bold mb-2 text-gray-50 uppercase">Game Over!</h2>
        <p class="text-5xl mb-4 font-extrabold text-gray-50">Your Score: {{ score }}/{{ total }}</p>

        <div class="flex justify-center space-x-4 pt-5">
          <button @click="restartGame"
            class="text-white bg-gradient-to-r from-green-400 via-green-500 to-green-600 hover:bg-gradient-to-br focus:ring-4 focus:outline-none focus:ring-green-300 dark:focus:ring-green-800 shadow-lg shadow-green-500/50 dark:shadow-lg dark:shadow-green-800/80 font-medium rounded-lg text-sm px-5 py-2.5 text-center me-2 mb-2">
            Play Again
          </button>
        </div>
      </div>

    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'

const questions = ref([])
const currentIndex = ref(0)
const currentQuestion = computed(() => questions.value[currentIndex.value])
const loading = ref(true)
const showAnswer = ref(false)
const isCorrect = ref(false)
const score = ref(0)
const total = ref(10) // Total questions
const gameOver = ref(false)

const fetchQuestions = async () => {
  loading.value = true
  try {
    const res = await axios.get(`https://opentdb.com/api.php?amount=${total.value}&type=multiple`)
    questions.value = res.data.results;
    console.log(res.data.results);
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

const decode = (text) => {
  const txt = document.createElement("textarea")
  txt.innerHTML = text
  return txt.value
}

const handleAnswer = (choice) => {
  showAnswer.value = true
  isCorrect.value = choice === currentQuestion.value.correct_answer
  if (isCorrect.value) score.value++
}

const nextQuestion = () => {
  showAnswer.value = false
  isCorrect.value = false
  currentIndex.value++

  if (currentIndex.value >= questions.value.length) {
    gameOver.value = true
  }
}

const restartGame = () => {
  score.value = 0
  currentIndex.value = 0
  showAnswer.value = false
  isCorrect.value = false
  gameOver.value = false
  fetchQuestions()
}

// Shuffle choices
const shuffledChoices = computed(() => {
  if (!currentQuestion.value) return []
  const choices = [
    currentQuestion.value.correct_answer,
    ...currentQuestion.value.incorrect_answers,
  ]
  return choices.sort(() => Math.random() - 0.5)
})

onMounted(fetchQuestions)
 
</script>

<style scoped>
.container-1 {
  background-image: url('/7290753.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-attachment: fixed;
}
</style>