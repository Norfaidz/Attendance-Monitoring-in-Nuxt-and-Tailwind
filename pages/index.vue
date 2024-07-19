<template>
    <div class="flex justify-center items-center min-h-screen bg-cyan-950 my-0">
      <div v-if="!eventEntered" class="p-6 bg-white shadow rounded-lg w-full max-w-md transition-opacity duration-500" :class="{'opacity-0': transitioning}">
        <h2 class="text-2xl font-bold mb-6 text-center">Enter Event Name</h2>
        <input v-model="eventName" @keyup.enter="enterEvent" placeholder="Event Name" class="border p-2 rounded w-full mb-4" />
        <button @click="enterEvent" class="bg-blue-500 text-white p-2 rounded w-full">Enter Event</button>
      </div>
      <div v-else class="p-6 bg-white shadow rounded-lg w-full max-w-3xl">
        <h2 class="text-2xl font-bold mb-6 text-center">Welcome to {{ eventName }}!!</h2>
        <div class="mb-6 flex justify-center">
          <input v-model="newpersonName" @keyup.enter="addperson" placeholder="Enter Person's Name" class="border p-2 rounded mr-2" />
          <button @click="addperson" class="bg-green-500 text-white p-2 rounded">Add Person</button>
        </div>
        <table id="attendance-table" class="min-w-full bg-white border border-gray-300">
          <thead>
            <tr>
              <th class="border border-gray-300 px-4 py-2">Name</th>
              <th class="border border-gray-300 px-4 py-2">Time Logged In</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="person in persons" :key="person.id" class="hover:bg-gray-100">
              <td class="border border-gray-300 px-4 py-2">{{ person.name }}</td>
              <td class="border border-gray-300 px-4 py-2">{{ person.timeLogged }}</td>
            </tr>
          </tbody>
        </table>
        <div class="flex justify-end mt-4">
          <button @click="finishAttendance" class="bg-red-500 text-white p-2 rounded">Done</button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref } from 'vue'
  import jsPDF from 'jspdf'
  import html2canvas from 'html2canvas'
  
  interface person {
    id: string
    name: string
    timeLogged: string
  }
  
  const persons = ref<person[]>([])
  const eventName = ref('')
  const eventEntered = ref(false)
  const transitioning = ref(false)
  const newpersonName = ref('')
  
  const enterEvent = () => {
    if (eventName.value) {
      transitioning.value = true
      setTimeout(() => {
        eventEntered.value = true
        transitioning.value = false
      }, 500)
    }
  }
  
  const addperson = () => {
    if (newpersonName.value) {
      const currentTime = new Date().toLocaleString()
      persons.value.push({
        id: `${persons.value.length + 1}`,
        name: newpersonName.value,
        timeLogged: currentTime,
      })
      newpersonName.value = ''
    }
  }
  
  const finishAttendance = async () => {
    if (confirm('Do you want to save the attendance as a PDF?')) {
      const element = document.getElementById('attendance-table')
      const canvas = await html2canvas(element)
      const imgData = canvas.toDataURL('image/png')
      const pdf = new jsPDF()
      const today = new Date().toISOString().split('T')[0]
      const pdfName = `${eventName.value}_${today}.pdf`
      pdf.addImage(imgData, 'PNG', 0, 0)
      pdf.save(pdfName)
      resetPage()
    }
    else{
        resetPage();
    }
  }
  
  const resetPage = () => {
    eventName.value = ''
    eventEntered.value = false
    persons.value = []
    transitioning.value = false
    newpersonName.value = ''
  }
  </script>