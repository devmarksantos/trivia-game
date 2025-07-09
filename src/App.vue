<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-gray-100">
    <h1 class="text-3xl font-semibold pb-5">🧠Trivia Quiz Game</h1>

    <div v-if="loading">
      <p class="text-gray-500">Loading question...</p>
    </div>

    <div v-else-if="currentQuestion" class="mt-5">
      <p class=" text-5xl font-bold pb-5" v-html="decode(currentQuestion.question)"></p>

      <div class="grid gap-2 mb-5">
        <button v-for="choice in shuffledChoices" :key="choice" @click="handleAnswer(choice)" :disabled="showAnswer"
          :class="[
            'px-4 py-2 rounded-md border text-left transition',
            showAnswer
              ? choice === currentQuestion.correct_answer
                ? 'bg-green-500 text-white border-green-500'
                : 'bg-red-500 text-white border-red-500'
              : 'bg-white hover:bg-gray-100 border-gray-300'
          ]" v-html="decode(choice)" />
      </div>

      <div v-if="showAnswer" class="mt-5 mb-4 items-center pt-5">

        <div v-if="isCorrect" class="bg-green-200 px-6 py-4 my-4 rounded-md text-lg flex items-center mx-auto mt-5">
          <svg viewBox="0 0 24 24" class="text-green-600 w-5 h-5 sm:w-5 sm:h-5 mr-3">
            <path fill="currentColor"
              d="M12,0A12,12,0,1,0,24,12,12.014,12.014,0,0,0,12,0Zm6.927,8.2-6.845,9.289a1.011,1.011,0,0,1-1.43.188L5.764,13.769a1,1,0,1,1,1.25-1.562l4.076,3.261,6.227-8.451A1,1,0,1,1,18.927,8.2Z">
            </path>
          </svg>
          <span class="text-green-800">Your answer is CORRECT.</span>
        </div>

        <div v-else class="bg-red-200 px-6 py-4 my-4 rounded-md text-lg flex items-center mx-auto mt-5">
          <svg viewBox="0 0 24 24" class="text-red-600 w-5 h-5 sm:w-5 sm:h-5 mr-3">
            <path fill="currentColor"
              d="M11.983,0a12.206,12.206,0,0,0-8.51,3.653A11.8,11.8,0,0,0,0,12.207,11.779,11.779,0,0,0,11.8,24h.214A12.111,12.111,0,0,0,24,11.791h0A11.766,11.766,0,0,0,11.983,0ZM10.5,16.542a1.476,1.476,0,0,1,1.449-1.53h.027a1.527,1.527,0,0,1,1.523,1.47,1.475,1.475,0,0,1-1.449,1.53h-.027A1.529,1.529,0,0,1,10.5,16.542ZM11,12.5v-6a1,1,0,0,1,2,0v6a1,1,0,1,1-2,0Z">
            </path>
          </svg>
          <span class="text-red-800"> Wrong answer! Correct answer is : <span
              v-html="decode(currentQuestion.correct_answer)"></span> </span>
        </div>
 
        <div class="flex justify-center space-x-4 pt-5">
          <button @click="nextQuestion" class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
            Next Question
          </button>
        </div>

      </div>
    </div>

    <div v-if="gameOver" class="mt-5 text-center">
      <h2 class="text-4xl font-bold mb-2">Game Over!</h2>
      <p class="text-4xl mb-4 font-extrabold">Your Score: {{ score }}/{{ total }}</p>

      <div class="flex justify-center space-x-4 pt-5">
        <button @click="restartGame" class=" px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700">
          Play Again
        </button>
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
    questions.value = res.data.results
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

const getChoiceClass = (choice) => {
  if (!showAnswer.value) return ''
  if (choice === currentQuestion.value.correct_answer) return 'correct'
  if (choice !== currentQuestion.value.correct_answer && choice === currentQuestion.value.userAnswer) return 'wrong'
  return ''
}
</script>