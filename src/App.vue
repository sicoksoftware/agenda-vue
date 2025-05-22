<template>
  <div class="flex h-screen bg-gray-100">
    <!-- Sidebar: Pacientes do dia -->
    <aside class="w-72 bg-white border-r border-gray-200 p-6 box-border flex flex-col">
      <h2 class="text-lg font-semibold mb-4 flex items-center gap-2">
        <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="icon icon-tabler icons-tabler-outline icon-tabler-clock"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M3 12a9 9 0 1 0 18 0a9 9 0 0 0 -18 0" /><path d="M12 7v5l3 3" /></svg>
        Agendamentos de hoje
      </h2>
      <ul class="flex-1 space-y-2">
        <li v-for="p in patientsOfDay" :key="p.id" class="flex items-center justify-between py-1 px-2 rounded hover:bg-gray-50">
          <div>
            <span class="block text-sm font-medium text-gray-800">
              {{ p.time }} <span class="text-gray-600">{{ p.name }}</span>
            </span>
            <span v-if="p.status" class="text-xs text-gray-400">{{ p.status }}</span>
          </div>

          <span :class="p.done ? 'text-green-500' : 'text-gray-300'">
          <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="icon icon-tabler icons-tabler-outline icon-tabler-circle-check"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M12 12m-9 0a9 9 0 1 0 18 0a9 9 0 1 0 -18 0" /><path d="M9 12l2 2l4 -4" /></svg>
          </span>
        </li>
      </ul>
    </aside>

    <!-- Main content -->
    <main class="flex-1 flex flex-col p-6">
      <!-- Header -->
      <header class="flex items-center justify-between mb-4 gap-4">
        <div class="flex items-center gap-2">
          <input type="text" placeholder="Busque cliente" class="border border-gray-300 rounded px-3 py-2 w-72 focus:outline-none focus:ring-2 focus:ring-blue-200" />
          <button class="bg-sky-600 text-white rounded px-3 py-2 text-lg flex items-center gap-2">
            <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="icon icon-tabler icons-tabler-outline icon-tabler-search"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M10 10m-7 0a7 7 0 1 0 14 0a7 7 0 1 0 -14 0" /><path d="M21 21l-6 -6" /></svg>
            <span>Pesquisar</span>
          </button>
        </div>

        <div class="flex items-center gap-2">
          <button class="bg-sky-600 text-white rounded px-3 py-2 text-lg flex items-center gap-2">
            <svg  xmlns="http://www.w3.org/2000/svg"  width="24"  height="24"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  stroke-width="2"  stroke-linecap="round"  stroke-linejoin="round"  class="icon icon-tabler icons-tabler-outline icon-tabler-calendar-plus"><path stroke="none" d="M0 0h24v24H0z" fill="none"/><path d="M12 8v4l3 3m6 -3l-3 3l-3 -3" /><path d="M12 8v4l-3 -3m-6 3l3 -3l3 3" /><path d="M12 8v-4l-3 3l3 -3z" /></svg>
            <span>Criar agendamento</span>
          </button>
        </div>
        
      </header>

      <!-- Agenda semanal -->
      <div class="overflow-x-auto bg-white rounded-lg shadow">
        <div class="flex items-center justify-between p-4">
          <div class="flex items-center gap-2">
            <button 
              class="px-3 py-2 text-lg flex items-center gap-2" 
              :class="currentView === 'day' ? 'bg-sky-600 text-white' : 'bg-gray-600 text-white'"
              @click="currentView = 'day'"
            >
              Dia
            </button>
            <button 
              class="px-3 py-2 text-lg flex items-center gap-2"
              :class="currentView === 'week' ? 'bg-sky-600 text-white' : 'bg-gray-600 text-white'"
              @click="currentView = 'week'"
            >
              Semana
            </button>
            <button 
              class="px-3 py-2 text-lg flex items-center gap-2"
              :class="currentView === 'month' ? 'bg-sky-600 text-white' : 'bg-gray-600 text-white'"
              @click="currentView = 'month'"
            >
              Mês
            </button>
          </div>
          <div class="flex gap-2 items-center">
            <button class="bg-sky-600 text-white rounded px-3 py-1 text-lg" @click="goToPrevious">&#8592;</button>
            <span>{{ currentDateDisplay }}</span>
            <button class="bg-sky-600 text-white rounded px-3 py-1 text-lg" @click="goToNext">&#8594;</button>
          </div>
        </div>

        <!-- Calendar Content -->
        <div class="p-4">
          <!-- Day View -->
          <div v-if="currentView === 'day'" class="space-y-4">
            <div v-for="hour in hours" :key="hour" class="flex gap-4">
              <div class="w-20 text-gray-600">{{ hour }}</div>
              <div class="flex-1">
                <div v-for="appointment in getAppointmentsForDay(selectedDate, hour)" 
                     :key="appointment.id"
                     :class="[appointment.color, 'p-2 rounded mb-2']">
                  {{ appointment.client_name }}
                </div>
              </div>
            </div>
          </div>

          <!-- Week View -->
          <div v-if="currentView === 'week'" class="grid grid-cols-7 gap-4">
            <div v-for="(date, index) in week" :key="index" class="border-r last:border-r-0">
              <div class="text-center font-semibold mb-2">{{ formatDate(date) }}</div>
              <div v-for="hour in hours" :key="hour" class="mb-4">
                <div class="text-xs text-gray-600 mb-1">{{ hour }}</div>
                <div v-for="appointment in getAppointmentsForDay(date, hour)" 
                     :key="appointment.id"
                     :class="[appointment.color, 'p-2 rounded text-sm mb-1']">
                  {{ appointment.client_name }}
                </div>
              </div>
            </div>
          </div>

          <!-- Month View -->
          <div v-if="currentView === 'month'" class="grid grid-cols-7 gap-4">
            <div v-for="(date, index) in monthDays" :key="index" 
                 class="border p-2 min-h-[100px]"
                 :class="{'bg-gray-50': !isSameMonth(date, selectedDate)}">
              <div class="text-center font-semibold mb-2">{{ date.getDate() }}</div>
              <div v-for="appointment in getAppointmentsForDay(date)" 
                   :key="appointment.id"
                   :class="[appointment.color, 'p-2 rounded text-sm mb-1']">
                {{ appointment.time }} - {{ appointment.client_name }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";

// Fake patients of the day
const patientsOfDay = [
  { id: 1, time: '08:00', name: 'Geovanna Silva', done: true },
  { id: 2, time: '09:00', name: 'João Paulo', done: true },
  { id: 3, time: '10:00', name: 'Maicon Guedes', done: false },
  { id: 4, time: '11:00', name: 'Leandro Amaral', done: false },
  { id: 5, time: '14:00', name: 'Júlia Santos', done: false },
  { id: 6, time: '16:00', name: 'Regina Marques', status: 'Aguardando há 4 min.' },
];

// Semana fake
const today = new Date();
const getWeek = (date) => {
  const start = new Date(date);
  start.setDate(start.getDate() - start.getDay()); // domingo
  return Array.from({ length: 7 }, (_, i) => {
    const d = new Date(start);
    d.setDate(start.getDate() + i);
    return d;
  });
};
const week = ref(getWeek(today));

const hours = Array.from({ length: 11 }, (_, i) => `${(i + 8).toString().padStart(2, '0')}:00`); // 08:00 às 18:00

const formatDate = (date) => {
  return date.toLocaleDateString('pt-BR', { day: '2-digit', month: '2-digit' });
};

// Fake agendamentos
const appointments = [
  { id: 1, client_name: 'Geovanna Silva', date: 2, time: '08:00', color: 'bg-blue-100 text-blue-700' },
  { id: 2, client_name: 'Maicon Guedes', date: 2, time: '09:00', color: 'bg-orange-100 text-orange-700' },
  { id: 3, client_name: 'Leandro Amaral', date: 2, time: '10:00', color: 'bg-blue-100 text-blue-700' },
  { id: 4, client_name: 'Júlia Santos', date: 2, time: '14:00', color: 'bg-blue-100 text-blue-700' },
  { id: 5, client_name: 'Regina Marques', date: 2, time: '16:00', color: 'bg-yellow-100 text-yellow-700' },
  { id: 6, client_name: 'Paula Neves', date: 3, time: '09:00', color: 'bg-pink-100 text-pink-700' },
  { id: 7, client_name: 'Bruno Santos', date: 4, time: '08:00', color: 'bg-purple-100 text-purple-700' },
  { id: 8, client_name: 'Maria José', date: 5, time: '09:00', color: 'bg-pink-100 text-pink-700' },
  { id: 9, client_name: 'José Luiz', date: 5, time: '10:00', color: 'bg-pink-100 text-pink-700' },
  { id: 10, client_name: 'Ricardo Ramos', date: 6, time: '17:00', color: 'bg-blue-100 text-blue-700' },
  { id: 11, client_name: 'Douglas Amaral', date: 6, time: '17:00', color: 'bg-orange-100 text-orange-700' },
  { id: 12, client_name: 'Cristiano Freitas', date: 6, time: '17:00', color: 'bg-yellow-100 text-yellow-700' },
];

// View state
const currentView = ref('week');
const selectedDate = ref(new Date());

// Computed properties for date display
const currentDateDisplay = computed(() => {
  if (currentView.value === 'day') {
    return selectedDate.value.toLocaleDateString('pt-BR', { 
      day: '2-digit', 
      month: '2-digit', 
      year: 'numeric' 
    });
  } else if (currentView.value === 'week') {
    const start = week.value[0];
    const end = week.value[6];
    return `${formatDate(start)} - ${formatDate(end)}`;
  } else {
    return selectedDate.value.toLocaleDateString('pt-BR', { 
      month: 'long', 
      year: 'numeric' 
    });
  }
});

// Month view helpers
const getMonthDays = (date) => {
  const year = date.getFullYear();
  const month = date.getMonth();
  const firstDay = new Date(year, month, 1);
  const lastDay = new Date(year, month + 1, 0);
  
  // Get the first day of the month (0-6, where 0 is Sunday)
  const firstDayIndex = firstDay.getDay();
  
  // Get the last day of the month
  const lastDayIndex = lastDay.getDate();
  
  // Create array of all days in the month
  const days = [];
  
  // Add days from previous month
  for (let i = firstDayIndex - 1; i >= 0; i--) {
    const d = new Date(year, month, -i);
    days.push(d);
  }
  
  // Add days from current month
  for (let i = 1; i <= lastDayIndex; i++) {
    const d = new Date(year, month, i);
    days.push(d);
  }
  
  // Add days from next month to complete the grid
  const remainingDays = 42 - days.length; // 6 rows * 7 days
  for (let i = 1; i <= remainingDays; i++) {
    const d = new Date(year, month + 1, i);
    days.push(d);
  }
  
  return days;
};

const monthDays = computed(() => getMonthDays(selectedDate.value));

const isSameMonth = (date1, date2) => {
  return date1.getMonth() === date2.getMonth() && 
         date1.getFullYear() === date2.getFullYear();
};

// Navigation functions
const goToPrevious = () => {
  if (currentView.value === 'day') {
    selectedDate.value.setDate(selectedDate.value.getDate() - 1);
    week.value = getWeek(selectedDate.value);
  } else if (currentView.value === 'week') {
    goToPreviousWeek();
  } else {
    selectedDate.value.setMonth(selectedDate.value.getMonth() - 1);
    week.value = getWeek(selectedDate.value);
  }
};

const goToNext = () => {
  if (currentView.value === 'day') {
    selectedDate.value.setDate(selectedDate.value.getDate() + 1);
    week.value = getWeek(selectedDate.value);
  } else if (currentView.value === 'week') {
    goToNextWeek();
  } else {
    selectedDate.value.setMonth(selectedDate.value.getMonth() + 1);
    week.value = getWeek(selectedDate.value);
  }
};

// Update the week navigation functions to also update selectedDate
const goToPreviousWeek = () => {
  const prev = new Date(week.value[0]);
  prev.setDate(prev.getDate() - 7);
  week.value = getWeek(prev);
  selectedDate.value = new Date(prev);
};

const goToNextWeek = () => {
  const next = new Date(week.value[0]);
  next.setDate(next.getDate() + 7);
  week.value = getWeek(next);
  selectedDate.value = new Date(next);
};

// Add a watch to update week when selectedDate changes
watch(selectedDate, (newDate) => {
  week.value = getWeek(newDate);
});

// Helper function to get appointments for a specific day and hour
const getAppointmentsForDay = (date, hour = null) => {
  const dayOfWeek = date.getDay(); // 0 = Sunday, 1 = Monday, etc.
  
  return appointments.filter(a => {
    const matchesDay = a.date === dayOfWeek;
    if (hour) {
      return matchesDay && a.time === hour;
    }
    return matchesDay;
  });
};
</script>

