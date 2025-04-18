<script setup lang="ts">
import { computed } from 'vue'
import {
  Chart as ChartJS,
  Title,
  RadialLinearScale,
  PointElement,
  LineElement,
  Filler,
  Tooltip,
  Colors,
} from 'chart.js'
import { Radar } from 'vue-chartjs'

ChartJS.register(RadialLinearScale, Title, PointElement, LineElement, Filler, Tooltip, Colors)
ChartJS.defaults.font.family = "'DM Sans', sans-serif"
ChartJS.defaults.font.size = 30

export interface Values {
  title: string
  value: number
}

const props = defineProps<{
  values: Values[]
}>()

const data = computed(() => {
  return {
    labels: props.values.map((e) => e.title),
    datasets: [
      {
        label: 'Average',
        data: props.values.map((e) => e.value),
      },
    ],
  }
})

const splitText = (text: string, maxLength: number): string[] => {
  const words = text.split(' ')
  const result: string[] = []
  let currentLine = ''

  for (const word of words) {
    if ((currentLine + ' ' + word).length <= maxLength) {
      currentLine = currentLine + ' ' + word
    } else {
      if (currentLine) {
        result.push(currentLine)
      }
      currentLine = word
    }
  }
  result.push(currentLine)
  return result
}

const pointLabels = (label: string): string[] => {
  return splitText(label, 30)
}

const options = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    r: {
      suggestedMin: 0,
      suggestedMax: 5,
      ticks: {
        stepSize: 1,
        font: {
          size: 14,
        },
      },
      pointLabels: {
        font: {
          size: 14,
        },
        callback: pointLabels,
      },
    },
  },
  plugins: {
    title: {
      display: true,
      text: 'Operational Excellence Assessment',
      font: {
        family: "'DM Sans', sans-serif",
        weight: 500,
        size: 18,
      },
    },
  },
}
</script>

<template>
  <div class="chart-container">
    <Radar :data="data" :options="options" />
  </div>
</template>

<style scoped>
.chart-container {
  background-color: #ffffff;
  min-width: 700px;
  min-height: 500px;
}

canvas {
  display: block;
}
</style>
