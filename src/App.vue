<script setup>
import { ref, onMounted } from 'vue'
import PocketBase from 'pocketbase'

const pb = new PocketBase('http://127.0.0.1:8090')
const hi = "Welcome to my"
let name = ref("City Explorer")
let number = ref(0)
let capitalCities = ref(["London", "Paris", "Berlin", "Madrid", "Kuala Lumpur"])
let records = ref([])
let newCity = ref("")
let isTrue = ref(false)
const solatAPI = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR01"
const matiAPI = "https://api.data.gov.my/data-catalogue?id=deaths&limit=10"
let prayerTime = ref({})
let mati = ref([])
let isLoading = ref(false)
let prayerTimeLoading = ref(false)
let deathDataLoading = ref(false)
const todos = ref("")
let editingId = ref(null)
let editingValue = ref("")


function increment() {
  if (number.value < 10) {
    number.value++;
  } else {
    number.value = 0;
  }
}

function startInterval() {
  setInterval(increment, 1000);
}

function addCity() {
  if (newCity.value.trim() !== '') {
    capitalCities.value.push(newCity.value);
    newCity.value = "";
  }
}

function removeCity(index) {
  capitalCities.value.splice(index, 1);
}

function addtoList(e) {
  if (e.key === "Enter" && newCity.value.trim() !== '') {
    capitalCities.value.push(newCity.value);
    newCity.value = "";
  }
}

function getSolat() {
  prayerTimeLoading.value = true;
  fetch(solatAPI)
    .then(result => result.json())
    .then(data => {
      prayerTime.value = data.prayerTime[0];
      console.log(prayerTime.value);
    })
    .catch(error => {
      console.error("Error fetching prayer times:", error);
    })
    .finally(() => {
      prayerTimeLoading.value = false;
    });
}

function getMati() {
  deathDataLoading.value = true;
  fetch(matiAPI)
    .then(result => result.json())
    .then(data => {
      mati.value = data;
      console.log(mati.value);
    })
    .catch(error => {
      console.error("Error fetching death data:", error);
    })
    .finally(() => {
      deathDataLoading.value = false;
    });
}

function addTodo() {
  if (todos.value.trim() !== '') {
    pb.collection('todos').create({ item: todos.value })
      .then(() => {
        todos.value = "";
        pb.collection('todos').getFullList().then(data => records.value = data)
      })
      .catch(error => {
        console.error("Error adding todo:", error);
      });
  }
}

function removetodo(id) {
  pb.collection('todos').delete(id)
    .then(() => {
      pb.collection('todos').getFullList().then(data => records.value = data)
    })
    .catch(error => {
      console.error("Error deleting todo:", error);
    });
}
onMounted(() => {
  getSolat();
  startInterval();
  pb.collection('todos').getFullList().then(data => records.value = data)
    
})
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-500 via-purple-500 to-pink-500 p-6">
    <div class="max-w-6xl mx-auto">
      <!-- Header -->
      <header class="mb-10 text-center">
        <h1 class="text-4xl font-bold text-white mb-2">{{ hi }} <span class="text-yellow-300">{{ name }}</span></h1>
        <div class="bg-white bg-opacity-20 rounded-full p-3 inline-block">
          <h2 class="text-2xl font-bold text-purple">{{ number }}</h2>
        </div>
      </header>
      
      <!-- Main Content -->
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        
        <!-- City List Section -->
        <div class="bg-white rounded-xl shadow-lg p-6 transform transition-all duration-300 hover:scale-105">
          <h2 class="text-2xl font-bold text-purple-800 mb-4">Capital Cities</h2>
          
          <div class="flex mb-4">
            <input 
              type="text" 
              v-model="newCity" 
              @keyup.enter="addCity"
              placeholder="Add a city" 
              class="flex-grow px-4 py-2 rounded-l-lg border-2 border-purple-300 focus:border-purple-500 focus:outline-none"
            />
            <button 
              @click="addCity" 
              class="bg-purple-600 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded-r-lg transition-colors duration-300"
            >
              Add
            </button>
          </div>
          
          <div class="max-h-64 overflow-y-auto pr-2">
            <transition-group name="list" tag="ul" class="space-y-2">
              <li 
                v-for="(city, index) in capitalCities" 
                :key="city"
                class="flex justify-between items-center bg-purple-100 rounded-lg p-3 animate-fade-in"
              >
                <span class="font-medium text-purple-800">{{ city }}</span>
                <button 
                  @click="removeCity(index)"
                  class="text-red-500 hover:text-red-700"
                >
                  ×
                </button>
              </li>
            </transition-group>
          </div>
          
          <div class="mt-4">
            <button 
              @click="isTrue = !isTrue" 
              class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg transition-colors duration-300 w-full"
            >
              Toggle City Pride
            </button>
            
            <div 
              v-if="isTrue" 
              class="mt-4 bg-gradient-to-r from-yellow-400 to-yellow-600 p-4 rounded-lg text-center animate-pulse"
            >
              <h3 class="text-xl font-bold text-white">England is my city!</h3>
            </div>
          </div>
        </div>
        
        <!-- Prayer Times Section -->
        <div class="bg-white rounded-xl shadow-lg p-6 transform transition-all duration-300 hover:scale-105">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-2xl font-bold text-teal-800">Prayer Times</h2>
            <button 
              @click="getSolat" 
              class="bg-teal-600 hover:bg-teal-700 text-white font-bold py-2 px-4 rounded-lg transition-colors duration-300 flex items-center"
              :disabled="prayerTimeLoading"
            >
              <span v-if="prayerTimeLoading">Loading...</span>
              <span v-else>Refresh</span>
            </button>
          </div>
          
          <div class="bg-gradient-to-r from-teal-500 to-emerald-500 rounded-lg shadow-md p-4">
            <div v-if="prayerTimeLoading" class="text-center py-8">
              <div class="inline-block animate-spin rounded-full h-8 w-8 border-4 border-white border-t-transparent"></div>
            </div>
            <table v-else class="w-full text-black">
              <thead>
                <tr>
                  <th class="py-2 px-1 text-center">Fajr</th>
                  <th class="py-2 px-1 text-center">Dhuhr</th>
                  <th class="py-2 px-1 text-center">Asr</th>
                  <th class="py-2 px-1 text-center">Maghrib</th>
                  <th class="py-2 px-1 text-center">Isha</th>
                </tr>
              </thead>
              <tbody>
                <tr class="text-center font-bold">
                  <td class="py-4 px-1 bg-white bg-opacity-20 rounded-lg m-1">{{ prayerTime.fajr || '—' }}</td>
                  <td class="py-4 px-1 bg-white bg-opacity-20 rounded-lg m-1">{{ prayerTime.dhuhr || '—' }}</td>
                  <td class="py-4 px-1 bg-white bg-opacity-20 rounded-lg m-1">{{ prayerTime.asr || '—' }}</td>
                  <td class="py-4 px-1 bg-white bg-opacity-20 rounded-lg m-1">{{ prayerTime.maghrib || '—' }}</td>
                  <td class="py-4 px-1 bg-white bg-opacity-20 rounded-lg m-1">{{ prayerTime.isha || '—' }}</td>
                </tr>
              </tbody>
            </table>
          </div>
          
          <!-- Death Data Section -->
          <div class="mt-6">
            <div class="flex justify-between items-center mb-4">
              <h2 class="text-2xl font-bold text-blue-800">Mortality Data</h2>
              <button 
                @click="getMati" 
                class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition-colors duration-300"
                :disabled="deathDataLoading"
              >
                <span v-if="deathDataLoading">Loading...</span>
                <span v-else>Load Data</span>
              </button>
            </div>
            
            <div class="bg-gradient-to-r from-blue-500 to-cyan-500 rounded-lg shadow-md p-4">
              <div v-if="deathDataLoading" class="text-center py-8">
                <div class="inline-block animate-spin rounded-full h-8 w-8 border-4 border-white border-t-transparent"></div>
              </div>
              <table v-else-if="mati.length > 0" class="w-full text-white">
                <thead>
                  <tr class="border-b border-white border-opacity-30">
                    <th class="py-2 text-left">Year</th>
                    <th class="py-2 text-right">Rate</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, index) in mati.slice(0, 5)" :key="index" 
                      class="border-b border-white border-opacity-10">
                    <td class="py-3 text-left">{{ item?.date || '—' }}</td>
                    <td class="py-3 text-right font-bold">{{ item?.rate || '—' }}</td>
                  </tr>
                </tbody>
              </table>
              <div v-else class="text-center py-4 text-white">
                Click the button to load mortality data.
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="bg-white rounded-xl shadow-lg p-6 mt-6">
  <h1 class="text-2xl font-bold text-blue-800 mb-4">PocketBase Records</h1>
  <ul class="divide-y divide-gray-200">
    <li 
      v-for="record in records" 
      :key="record.id" 
      class="py-3 flex justify-between items-center"
    >
      <span class="text-gray-600">{{ record.item }}</span>
      <span><button 
                  @click="removetodo(record.id)"
                  class="text-red-500 hover:text-red-700"
                >
                  ×
                </button></span>

    </li>
  </ul>
  <div>
    <input v-model="todos" type="text" placeholder="Add a new todo" class="border border-gray-300 rounded-md p-2 mt-4 w-full" @keyup.enter="addTodo">
    <button @click="addTodo" class="mt-2 bg-blue-500 text-white rounded-md px-4 py-2">Add Todo</button>
  </div>
   

  </div>
</div>
<div>
      <!-- Footer -->
      <footer class="mt-10 text-center text-white text-opacity-70">
        <p>© 2025 City Explorer | Data courtesy of e-solat.gov.my and data.gov.my</p>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.animate-fade-in {
  animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>