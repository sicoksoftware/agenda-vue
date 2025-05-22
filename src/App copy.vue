<template>
  <div class="bg-red-500">
    <h1>teste</h1>
  </div>
  <div class="flex h-screen bg-gray-100">
    <!-- Painel lateral -->
    <aside class="w-64 bg-white border-r border-gray-200 p-6 box-border">
      <h2 class="text-lg font-semibold mb-4">Compromissos do Dia</h2>
      <ul class="space-y-3">
        <li v-for="a in appointments.value.filter(ap => ap.date === week.value[0])" :key="a.id" class="text-base">
          <span class="font-bold text-blue-700">{{ a.start_time }}</span> - {{ a.client_name }}
        </li>
      </ul>
    </aside>

    <!-- Conteúdo principal -->
    <main class="flex-1 flex flex-col p-6">
      <header class="flex items-center justify-center mb-4 gap-4">
        <button @click="goToPreviousWeek" class="bg-blue-700 text-white rounded px-3 py-1 text-lg">&#8592;</button>
        <span class="text-lg font-medium">
          {{ formatDate(week.value[0]) }} - {{ formatDate(week.value[6]) }}
        </span>
        <button @click="goToNextWeek" class="bg-blue-700 text-white rounded px-3 py-1 text-lg">&#8594;</button>
      </header>
      <div class="overflow-x-auto bg-white rounded-lg shadow">
        <table class="min-w-full border-collapse">
          <thead>
            <tr>
              <th class="w-20"></th>
              <th v-for="date in week.value" :key="date" class="py-2 px-4 border border-gray-200 text-center text-base font-semibold text-gray-700">{{ formatDate(date) }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="hour in hours" :key="hour">
              <td class="bg-gray-50 font-bold w-20 text-center border border-gray-200">{{ hour }}</td>
              <td v-for="date in week.value" :key="date" class="border border-gray-200 min-h-12 align-top">
                <div v-for="a in getAppointmentsByHour(date, hour)" :key="a.id" class="bg-blue-100 text-blue-700 rounded px-2 py-1 mb-1 cursor-pointer text-sm hover:bg-blue-200 transition" @click="openEditModal(a)">
                  {{ a.client_name }}
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </main>
  </div>
  <EditModal v-if="showModal.value" :appointment="selectedAppointment.value" @close="showModal.value = false" />
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import EditModal from './components/EditModal.vue'

const today = new Date()
const weekStart = ref(new Date(today.setDate(today.getDate() - today.getDay())))
const appointments = ref([])
const showModal = ref(false)
const selectedAppointment = ref(null)

const hours = Array.from({ length: 12 }, (_, i) => `${(i + 8).toString().padStart(2, '0')}:00`) // 08:00 às 19:00

const week = computed(() =>
  Array.from({ length: 7 }, (_, i) => {
    const date = new Date(weekStart.value)
    date.setDate(date.getDate() + i)
    return date.toISOString().slice(0, 10)
  })
)

function formatDate(date) {
  return new Date(date).toLocaleDateString('pt-BR', {
    weekday: 'short',
    day: '2-digit',
    month: '2-digit',
  })
}

function getAppointmentsByHour(date, hour) {
  return appointments.value.filter(a => a.date === date && a.start_time.startsWith(hour))
}

function goToPreviousWeek() {
  weekStart.value.setDate(weekStart.value.getDate() - 7)
  loadAppointments()
}

function goToNextWeek() {
  weekStart.value.setDate(weekStart.value.getDate() + 7)
  loadAppointments()
}

function openEditModal(appointment) {
  selectedAppointment.value = { ...appointment }
  showModal.value = true
}

async function loadAppointments() {
  const baseDate = new Date(weekStart.value)
  appointments.value = []
  for (let i = 0; i < 7; i++) {
    const date = new Date(baseDate)
    date.setDate(baseDate.getDate() + i)
    const isoDate = date.toISOString().slice(0, 10)
    appointments.value.push(
      { id: i * 3 + 1, client_name: 'Cliente A', date: isoDate, start_time: '08:00', end_time: '09:00' },
      { id: i * 3 + 2, client_name: 'Cliente B', date: isoDate, start_time: '10:00', end_time: '11:00' },
      { id: i * 3 + 3, client_name: 'Cliente C', date: isoDate, start_time: '14:00', end_time: '15:00' }
    )
  }
}

async function updateAppointment(appointmentData) {
 
}

onMounted(loadAppointments)
</script>