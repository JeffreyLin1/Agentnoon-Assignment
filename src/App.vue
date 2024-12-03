<script setup>
    import { ref, onMounted } from 'vue';
    import * as d3 from 'd3';
    import Tree from './components/tree.vue'; 
    import '@/assets/main.css';


    const treeData = ref(null);


    const csvFile = ref('https://docs.google.com/spreadsheets/d/e/2PACX-1vR-wJZpg2d_sAskHkxoR5ZFW7eFtTeljRcaiVWtoB5foj0Ac2_0QT18HXDgwm2C4eQbFHC-Zg9dEABs/pub?gid=0&single=true&output=csv');


    const userInput = ref(csvFile.value);

    async function loadCSV(url) {
      try {
 
        const response = await fetch(url);
        const text = await response.text();


        const csvData = d3.csvParse(text);

        console.log('Parsed CSV Data:', csvData);

        csvData.forEach(row => {
          row["Employee Id"] = row["Employee Id"].trim();
          row.Manager = row.Manager ? row.Manager.trim() : null;
        });


        const root = d3.stratify()
          .id((d) => d["Employee Id"]) 
          .parentId((d) => d["Manager"]) 
          (csvData);

        console.log("Processed Hierarchy Data:", root.descendants()[0]);
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
    <div
        class="min-w-screen bg-gray-100 flex flex-col items-center justify-center p-4"
    >
        <!-- Title -->

        <!-- URL Input and Button -->
        <div
            style="
                margin-bottom: 40px;
                margin-top:80px;
                display: flex;
                flex-direction: column;
                align-items: center;
                width: 80%;
                transform: translateX(10%);
                
            "
        >
            <img
                src="/agentnoonIcon.png"
                width="15%"
                height="15%"
                
            />
            <label
                for="csv-url"
                style="
                    
                    font-size: 30px;
                    font-weight: bold;
                    color: #000;
                    margin-bottom: 10px;
                    font-family: sans-serif;
                "
            >
            Create your organization diagram in 
            <span style="color: #004dec;text-decoration: underline;font-weight:bold;">seconds</span>.
            </label>
            </div>
            <div style="text-align: center; font-size: 16px; margin-top: 10px; color: #666;">
            <label
                for="csv-url"
                style="
                    display: block;
                    font-size: 18px;
                    font-weight: bold;
                    color: #333;
                    font-family: sans-serif;
                    margin-bottom: 10px;
                    transform: translateX(-1%);
                "
            >
                Enter CSV URL:
            </label>
            <div style="margin-bottom: 20px; position: relative; display: inline-block; width: 80%; ">
            <input
                id="csv-url"
                type="text"
                v-model="userInput"
                placeholder="Enter CSV URL here..."
                style="
                    width: 300px;
                    padding: 10px 45px 10px 5px;
                    border: 2px solid #ccc;
                    border-radius: 8px;
                    font-size: 14px;
                    color: #555;
                    margin-bottom: 15px;
                    
                "
                
            />
            <button
                @click="updateCSV"
                style="
                    padding: 8px 15px;
                    background-color: white;
                    color: #004dec;
                    font-size: 14px;
                    transform: translateX(-100%);
                    font-weight: bold;
                    border: 2px solid #ccc;
                    border-radius: 8px;
                    cursor: pointer;
                    transition: background-color 0.3s ease;
                "
                @mouseover="(e) => e.target.style.backgroundColor = '#929ac4'"
                @mouseout="(e) => e.target.style.backgroundColor = 'white'"
            >
            >
            </button>
          </div>
        </div>
        <div>
            <tree v-if="treeData" :data="treeData" />
        </div>

        <!-- Tree Component -->
    </div>
</template>
