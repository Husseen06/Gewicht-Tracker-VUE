<script setup>
import { ref, shallowRef, computed, watch, nextTick } from 'vue'
import Chart from 'chart.js/auto'

// OOP: WeightEntry class
class WeightEntry {
  constructor(weight, date = Date.now()) {
    this.weight = Number(weight)
    this.date = date
  }
}

// OOP: WeightTracker class
class WeightTracker {
  constructor() {
    const saved = JSON.parse(localStorage.getItem('weights')) || []
    this.entries = saved.map(e => new WeightEntry(e.weight, e.date))
  }

  add(weight) {
    const entry = new WeightEntry(weight)
    this.entries.push(entry)
    this.save()
  }

  delete(index) {
    this.entries.splice(index, 1)
    this.save()
  }

  getCurrent() {
    return this.entries.slice().sort((a, b) => b.date - a.date)[0] || new WeightEntry(0)
  }

  save() {
    localStorage.setItem('weights', JSON.stringify(this.entries))
  }
}

const tracker = ref(new WeightTracker())

const weights = computed(() => tracker.value.entries)
const weightInput = ref(0)
const weightChartEl = ref(null)
const weightChart = shallowRef(null)

const currentWeight = computed(() => tracker.value.getCurrent())

const addWeight = () => {
  if (weightInput.value) {
    tracker.value.add(weightInput.value)
    weightInput.value = ''
  }
}

const deleteWeight = (index) => {
  tracker.value.delete(index)
}

watch(weights, (newWeights) => {
  const ws = [...newWeights]
  if (weightChart.value) {
    weightChart.value.data.labels = ws
      .sort((a, b) => a.date - b.date)
      .map(weight => new Date(weight.date).toLocaleDateString('nl-NL'))
      .slice(-7)
    weightChart.value.data.datasets[0].data = ws
      .sort((a, b) => a.date - b.date)
      .map(weight => weight.weight)
      .slice(-7)
    weightChart.value.update()
    return
  }
  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: ws
          .sort((a, b) => a.date - b.date)
          .map(weight => new Date(weight.date).toLocaleDateString('nl-NL')),
        datasets: [
          {
            label: 'Gewicht',
            data: ws
              .sort((a, b) => a.date - b.date)
              .map(weight => weight.weight),
            backgroundColor: 'rgba(255, 105, 180, 0.2)',
            borderColor: 'rgba(255, 105, 180, 1)',
            borderWidth: 1,
            fill: true
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          tooltip: {
            callbacks: {
              label: function(context) {
                return context.dataset.label + ': ' + context.parsed.y + 'kg'
              }
            }
          }
        }
      }
    })
  })
}, { deep: true })
</script>

<template>
  <div id="app" class="scrollbar-none">
    <div class="app-card">
      <!-- Header -->
      <header class="mb-8">
        <h1 class="text-4xl font-extrabold text-pink-400 tracking-tight text-center drop-shadow">Gewicht Tracker</h1>
      </header>

      <!-- Current Weight Section -->
      <section class="flex flex-col items-center justify-center mb-8">
        <div class="flex flex-col items-center justify-center w-40 h-40 bg-gray-900 rounded-full shadow-lg border-4 border-pink-400 mb-4">
          <span class="text-5xl font-bold text-pink-300 mb-1">{{ currentWeight.weight }}</span>
          <small class="text-gray-400 italic">Huidig gewicht (kg)</small>
        </div>
        <form
          @submit.prevent="addWeight"
          class="flex flex-row w-full gap-3 mt-4"
        >
          <input
            type="number"
            step="0.1"
            v-model="weightInput"
            class="flex-1 p-3 rounded-lg text-lg outline-none border-none bg-gray-700 text-white placeholder-gray-400 focus:ring-2 focus:ring-pink-400 transition"
            placeholder="Voer gewicht in (kg)"
          />
          <input
            type="submit"
            value="Toevoegen"
            class="bg-pink-500 hover:bg-pink-600 text-white font-bold text-lg px-4 py-2 rounded-lg cursor-pointer transition"
          />
        </form>
      </section>

      <!-- Chart Section -->
      <section class="w-full flex flex-col items-center mb-8 bg-gray-800 rounded-xl p-4">
        <h2 class="mb-4 text-xl text-pink-300 font-semibold text-center">Afgelopen week</h2>
        <div class="bg-gray-900 p-4 rounded-xl shadow-lg w-full flex justify-center">
          <canvas ref="weightChartEl" class="w-full h-56"></canvas>
        </div>
      </section>

      <!-- History Section -->
      <section class="w-full flex flex-col items-center">
        <h2 class="text-xl font-semibold mb-4 text-pink-300 text-center">Gewicht geschiedenis</h2>
        <div class="w-full">
          <ul class="space-y-2 w-full">
            <li
              v-for="(weight, index) in weights"
              :key="index"
              class="flex flex-row items-center justify-between bg-gray-700 rounded-xl shadow p-3 even:bg-gray-800 hover:bg-gray-600 transition"
            >
              <div>
                <span class="font-bold text-pink-200 text-lg">{{ weight.weight }}kg</span>
                <small class="block text-gray-400 text-xs italic">
                  {{ new Date(weight.date).toLocaleDateString('nl-NL') }}
                </small>
              </div>
              <button
                @click="deleteWeight(index)"
                class="px-3 py-1 bg-pink-500 hover:bg-pink-600 text-white text-xs rounded transition"
              >
                Verwijderen
              </button>
            </li>
          </ul>
          <div v-if="weights.length === 0" class="text-center text-gray-400 mt-8">
            <p>Voer je eerste gewicht in om te beginnen!</p>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<style>
/* Hide scrollbar for all browsers */
body,
#app,
.scrollbar-none {
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE 10+ */
}
body::-webkit-scrollbar,
#app::-webkit-scrollbar,
.scrollbar-none::-webkit-scrollbar {
  display: none; /* Chrome/Safari/Webkit */
}
</style>