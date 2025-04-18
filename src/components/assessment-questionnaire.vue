<script setup lang="ts">
import bestPracticesJson from './data/best-practices.json'
import chart, { type Values } from '@/components/result-chart.vue'
import { computed, type ComputedRef, ref } from 'vue'

interface bestPractices {
  area: string
  area_link: string
  questions: {
    question: string
    question_link: string
    best_practices: {
      best_practice: string
      best_practice_link: string
      risk: string
      value?: number
    }[]
  }[]
}

const test: bestPractices[] = bestPracticesJson

const riskLevel = ref(['high'])

const options = ref([
  { text: 'high', value: ['high'] },
  { text: 'high & medium', value: ['high', 'medium'] },
  { text: 'high, medium & low', value: ['high', 'medium', 'low'] },
])

const resultsByArea = ref([
  {
    title: 'No Data',
    value: 0,
  },
])
const resultsByQuestion = ref([
  {
    title: 'No Data',
    value: 0,
  },
])

const calculateAverage = (arr: number[]) =>
  arr.reduce((sum: number, value: number) => sum + value, 0) / arr.length

const assessment: ComputedRef<bestPractices[]> = computed(() =>
  test
    .map((a) => ({
      area: a.area,
      area_link: a.area_link,
      questions: a.questions
        .map((q) => ({
          question: q.question,
          question_link: q.question_link,
          best_practices: q.best_practices
            .filter((b) => riskLevel.value.includes(b.risk))
            .map((b) => ({
              best_practice: b.best_practice,
              best_practice_link: b.best_practice_link,
              risk: b.risk,
              value: 0,
            })),
        }))
        .filter((q) => q.best_practices.length != 0),
    }))
    .filter((a) => a.questions.length != 0),
)

function generateCharts() {
  const perQuestion: Values[] = []
  assessment.value.forEach((area) => {
    area.questions.forEach((question) => {
      perQuestion.push({
        title: question.question,
        value: calculateAverage(question.best_practices.map((item) => Number(item.value))),
      })
    })
  })
  resultsByQuestion.value = perQuestion

  const perArea: Values[] = []
  assessment.value.forEach((area) => {
    perArea.push({
      title: area.area,
      value: calculateAverage(
        area.questions.flatMap((item) => item.best_practices.map((item2) => Number(item2.value))),
      ),
    })
  })
  resultsByArea.value = perArea
}
</script>

<template>
  <h1>Operational Excellence Assessment</h1>
  <p>
    Operational excellence is a commitment to build software correctly while consistently delivering
    a great customer experience. This assessment is based on the AWS Well-Architected Framework
    <a
      href="https://docs.aws.amazon.com/wellarchitected/latest/framework/a-operational-excellence.html"
      target="_blank"
      >operational excellence pillar</a
    >
    that contains best practices for organizing your team, designing your workload, operating it at
    scale, and evolving it over time.
  </p>
  <h2>Assessment</h2>
  <p>
    Every best practice has a level of risk exposed if it’s not established. Please select the
    level(s) you want to assess:
    <select v-model="riskLevel">
      <option v-for="option in options" :value="option.value" :key="option.text">
        {{ option.text }}
      </option>
    </select>
  </p>
  <template v-for="bestPractice in assessment" :key="bestPractice.area">
    <h3>{{ bestPractice.area }}</h3>
    <template v-for="question in bestPractice.questions" :key="question.question">
      <h4>{{ question.question }}</h4>
      <div class="assessment-grid">
        <template
          v-for="best_practice in question.best_practices"
          :key="best_practice.best_practice"
        >
          <div>
            <p>
              <a :href="best_practice.best_practice_link" target="_blank">{{
                best_practice.best_practice
              }}</a>
            </p>
          </div>
          <div>
            <p>{{ best_practice.risk }}</p>
          </div>
          <div class="assessment">
            <select :id="best_practice.best_practice" v-model="best_practice.value">
              <option value="0" disabled>Choose one</option>
              <option value="1">1 - Major deficits, immediate need for action</option>
              <option value="2">2 - Deficits, immediate need for action</option>
              <option value="3">3 - Individual deficits, medium-term need for action</option>
              <option value="4">4 - Good starting position, little need for action</option>
              <option value="5">5 - Very good starting position, no need for action</option>
            </select>
          </div>
        </template>
      </div>
    </template>
  </template>
  <h2>Summary</h2>
  <button @click="generateCharts" type="button">Generate summaries</button>
  <h3>Summary by Area</h3>
  <p>
    <chart :values="resultsByArea" />
  </p>
  <h3>Summary by Question</h3>
  <chart :values="resultsByQuestion" />
</template>

<style scoped>
h1 {
  margin-bottom: 25px;
  line-height: 56px;
}

h2 {
  margin-top: 40px;
  margin-bottom: 20px;
}

h3 {
  margin-top: 30px;
  margin-bottom: 20px;
}

h1 {
  font-size: 45px;
}

h2 {
  font-size: 34px;
}

h3 {
  font-size: 28px;
}

h1,
h3,
h4 {
  font-weight: 500;
}

h2 {
  font-weight: 300;
}

h4 {
  font-weight: 700;
}

.assessment-grid {
  display: grid;
  grid-auto-rows: min-content;
  grid-template-columns: 2fr 0.2fr;
  padding: 12px 12px 12px 0;
}

.assessment {
  grid-column: span 2;
}

.assessment-grid .assessment {
  margin-bottom: 10px;
}

@media (min-width: 768px) {
  .assessment-grid {
    display: grid;
    grid-auto-rows: min-content;
    grid-template-columns: 2fr 0.2fr 1fr;
    padding-left: 12px;
  }

  .assessment {
    grid-column: 3/3;
  }
}

button {
  border-color: rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  border-bottom-style: solid;
  border-bottom-width: 1px;
  padding: 10px 16px;
  cursor: pointer;
  background-color: #d9edfc;
}

button:hover {
  background-color: #37a2eb;
  color: white;
}

select {
  margin-left: 5px;
  border-radius: 6px;
  border-bottom-style: solid;
  border-bottom-width: 1px;
  padding: 1px 2px;
}
</style>
