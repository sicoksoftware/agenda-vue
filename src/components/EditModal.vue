<template>
    <div class="fixed inset-0 bg-black bg-opacity-40 flex justify-center items-center z-50">
      <div class="bg-white p-4 rounded w-96">
        <h3 class="text-lg font-semibold mb-2">Editar Agendamento</h3>
        <form @submit.prevent="$emit('salvar', local)">
          <label class="block mb-2">
            Cliente:
            <input v-model="local.client_name" class="border p-1 w-full" />
          </label>
          <label class="block mb-2">
            Serviço:
            <input v-model="local.service" class="border p-1 w-full" />
          </label>
          <label class="block mb-2">
            Data:
            <input type="date" v-model="local.date" class="border p-1 w-full" />
          </label>
          <label class="block mb-2">
            Hora Início:
            <input type="time" v-model="local.start_time" class="border p-1 w-full" />
          </label>
          <label class="block mb-2">
            Hora Fim:
            <input type="time" v-model="local.end_time" class="border p-1 w-full" />
          </label>
          <div class="flex justify-between mt-4">
            <button type="button" @click="$emit('fechar')" class="text-gray-600">Cancelar</button>
            <button class="bg-blue-500 text-white px-4 py-1 rounded" type="submit">Salvar</button>
          </div>
        </form>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, watch } from 'vue'
  
  const props = defineProps(['agendamento'])
  const emits = defineEmits(['salvar', 'fechar'])
  
  const local = ref({ ...props.agendamento })
  
  watch(() => props.agendamento, (v) => {
    local.value = { ...v }
  })
  </script>
  