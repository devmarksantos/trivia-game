<template>
  <div class="container">
    <h1>🧠 Trivia Quiz Game</h1>

    <div v-if="loading">Loading question...</div>

    <div v-else-if="currentQuestion">
      <p v-html="decode(currentQuestion.question)"></p>

      <div class="choices">
        <button v-for="choice in shuffledChoices" :key="choice" @click="handleAnswer(choice)" :disabled="showAnswer"
          :class="getChoiceClass(choice)" v-html="decode(choice)" />
      </div>

      <div v-if="showAnswer">
        <p v-if="isCorrect">✅ Correct!</p>
        <p v-else>❌ Wrong. Correct: <strong v-html="decode(currentQuestion.correct_answer)"></strong></p>
        <button @click="nextQuestion">Next Question</button>
      </div>
    </div>

    <div v-if="gameOver">
      <h2>🎉 Game Over!</h2>
      <p>Your Score: {{ score }}/{{ total }}</p>
      <button @click="restartGame">Play Again</button>
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
const total = ref(5) // Total questions
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

<style scoped>
.container { 
  margin: 3rem auto;
  padding: 2rem;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 0 12px rgba(0, 0, 0, 0.1);
  text-align: center;
}

button {
  margin: 0.5rem;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
}

.choices {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.correct {
  background-color: #4CAF50;
  color: white;
}

.wrong {
  background-color: #f44336;
  color: white;
}
</style>
