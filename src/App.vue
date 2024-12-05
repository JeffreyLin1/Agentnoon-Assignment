<script setup>
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';
import Tree from './components/tree.vue'; 
import '@/assets/main.css';

const treeData = ref(null);
const csvFile = ref(
  'https://docs.google.com/spreadsheets/d/e/2PACX-1vR-wJZpg2d_sAskHkxoR5ZFW7eFtTeljRcaiVWtoB5foj0Ac2_0QT18HXDgwm2C4eQbFHC-Zg9dEABs/pub?gid=0&single=true&output=csv'
);
const userInput = ref(csvFile.value);


async function loadCSV(url) {
  try {
    const response = await fetch(url);
    const text = await response.text();
    const csvData = d3.csvParse(text);

    csvData.forEach((row) => {
      row['Employee Id'] = row['Employee Id'].trim();
      row.Manager = row.Manager ? row.Manager.trim() : null;
    });

    const root = d3
      .stratify()
      .id((d) => d['Employee Id'])
      .parentId((d) => d['Manager'])(csvData);

    treeData.value = root;
  } catch (error) {
    console.error('Error fetching or processing CSV data:', error);
    alert('Failed to load CSV. Please check the URL.');
  }
}

onMounted(() => {
  loadCSV(csvFile.value);
});

function updateCSV() {
  csvFile.value = userInput.value;
  loadCSV(csvFile.value);
}

</script>

<template>
    <div class="min-h-screen flex flex-col items-center justify-start pt-8"
    style="background-color: #e6eff0;">

      <div class="mb-10 text-center" >
        <img src="/agentnoonIcon.png" alt="Logo" class="w-70 h-20 mx-auto" />
        <h1 class="text-2xl font-bold text-gray-800 mt-4">
          Create your organization diagram in
          <span class="text-blue-600 italic">seconds</span>
        </h1>
      </div>
  

      <div class="flex flex-col items-center mb-8 w-80">
        <label
          for="csv-url"
          class="block text-lg font-semibold text-gray-700 mb-2"
        >
          Enter CSV URL:
        </label>
        <div class="flex w-full">
          <input
            id="csv-url"
            type="text"
            v-model="userInput"
            placeholder="Enter CSV URL here..."
            class="flex-1 p-2 border border-gray-300 rounded-l-lg focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
          <button
            @click="updateCSV"
            class="px-4 bg-blue-600 text-white font-semibold rounded-r-lg hover:bg-blue-700 transition"
          >
            >
          </button>
        </div>
      </div>
  

      <div class="rounded-lg w-full max-w-8xl h-[140vh] overflow-auto">
        <tree v-if="treeData" :data="treeData" />
      </div>
    </div>
  </template>
  