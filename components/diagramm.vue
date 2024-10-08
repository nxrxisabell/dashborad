<template>
 <div v-if="auswertung" style="max-width: 1300px !important;max-height: 300px; height: 300px;">
  <v-btn @click="filterData('7days')">Last 7 Days </v-btn>
  <v-btn @click="filterData('1month')">Last Month </v-btn>
  <v-btn @click="filterData('3months')">Last 3 Month</v-btn>
  <Line
      :data="chartData"
      :options="chartOptions"
    />
 
  </div>
</template>
<script setup lang="ts">
import { Line } from 'vue-chartjs'
import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend,
    scales,
    TimeScale
    } from 'chart.js'
import'chartjs-adapter-dayjs';
    
    ChartJS.register(
        CategoryScale,
        LinearScale,
        PointElement,
        LineElement,
        Title,
        Tooltip,
        Legend,
        TimeScale
    )
    
    const props = defineProps(['auth'])
    const {data} = await useFetch("https://eu.iot-api.milesight-iot.com/v1/devices/0f153e4e-400e-42a4-a3fb-a5373d68b2be/records?pageSize=1000&order=desc", {method: "get", headers: {Authorization:`Bearer ${props.auth.jws}`, tokenid: props.auth.tokenid}}) 
    const auswertung = data.value.data
    const price = await useFetch("https://www.fingrid.fi/api/graph/dataset?configurationItems%5B0%5D.variableId=244&configurationItems%5B1%5D.variableId=106&start=2024-09-26+00%3A00%3A00&end=2024-10-13+23%3A59%3A59")
    
    const filteredData = auswertung.filter((item, index, arr) => {
    return index === 0 || item.battery !== arr[index - 1].battery;
});

    const labels = filteredData.map(item => new Date(item.ts * 1000));
    const batteryData = filteredData.map(item => item.battery);

    const truncateToHour = (date) => {
    const newDate = new Date(date);
    newDate.setMinutes(0, 0, 0); // Minuten, Sekunden und Millisekunden auf 0 setzen
    return newDate.getTime(); // Zeitstempel in Millisekunden zurückgeben
};

const labelsTruncated = labels.map(label => truncateToHour(label));

const priceDataTruncated = price.data.value[0].Values.map(item => ({
    ts: truncateToHour(new Date(item.Timestamp)), // Zeitstempel truncieren
    value: item.Value,
}));
console.log(priceDataTruncated)

// Preis-Daten filtern
const priceFilteredData = priceDataTruncated.filter(item =>
    labelsTruncated.includes(item.ts)
);
   console.log(priceFilteredData)
const filteredPriceData = priceFilteredData.map(item => item.value);
console.log(filteredPriceData)

    const chartData = ref({
        labels: labelsTruncated,
  datasets: [
    {
      label: 'Battery',
      backgroundColor: '#f87979',
      data: batteryData,
      yAxisID: 'y',

    },
    {label: 'Price',
        backgroundColor: '#g86786',
        data: filteredPriceData,
        yAxisID: 'y1',
    }
  ],
})
const chartOptions = ref({
  responsive: true,
  maintainAspectRatio: false,

  scales: {
    x: {
      type: 'time',
      time: {
        unit: 'day',
      },
      title: {
        display: true,
        text: 'Day',
      },},

    y: {
          title: {
            display: true,
            text: 'Batterie (%)',
        },
        position: 'left',
          min: 0,
          max: 100,
        },
        y1: {
            title: {
                display: true,
                text: 'Price (€)',
            },
            position: 'right',
      
  
        }

  }
})








</script>