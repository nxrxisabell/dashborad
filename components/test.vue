<template>
  <div>
    <button @click="filterData('7days')">Letzte 7 Tage</button>
    <button @click="filterData('1month')">Letzter Monat</button>
    <button @click="filterData('3months')">Letzte 3 Monate</button>
    <button @click="filterData('all')">Alle Daten</button>

    <canvas ref="chartCanvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import {
  Chart,
  CategoryScale,
  LinearScale,
  LineController,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  scales,
  TimeScale,
} from "chart.js";
Chart.register(
  CategoryScale,
  LinearScale,
  LineController,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  scales,
  TimeScale
);
import "chartjs-adapter-dayjs";
const props = defineProps(["auth"]);
const { data } = await useFetch(
  "https://eu.iot-api.milesight-iot.com/v1/devices/0f153e4e-400e-42a4-a3fb-a5373d68b2be/records?pageSize=1000&order=desc",
  {
    method: "get",
    headers: {
      Authorization: `Bearer ${props.auth.jws}`,
      tokenid: props.auth.tokenid,
    },
  }
);

const priceData = await useFetch(
  "https://www.fingrid.fi/api/graph/dataset?configurationItems%5B0%5D.variableId=244&configurationItems%5B1%5D.variableId=106&start=2024-09-26+00%3A00%3A00&end=2024-10-13+23%3A59%3A59"
);

const auswertung = ref([data.value.data]); // Dein ursprüngliches Datenset hier
const price = ref([priceData.data.value[0].Values]); // Preisdaten hier
const chartData = ref({
  labels: [],
  datasets: [
    {
      label: "Batterie",
      backgroundColor: "#f87979",
      data: [],
    },
    {
      label: "Preis",
      backgroundColor: "#f86786",
      data: [],
    },
  ],
});
const chart = ref(null);

const createChart = () => {
  const ctx = document.querySelector("canvas").getContext("2d");
  chart.value = new Chart(ctx, {
    type: "line",
    data: chartData.value,
    options: {
      responsive: true,
      scales: {
        x: {
          type: "time",
          time: {
            unit: "hour",
          },
        },
        y: {
          min: 0,
          max: 100,
        },
      },
    },
  });
};

const filterData = (period) => {
  const now = new Date();
  let filteredAuswertung = auswertung.value;

  switch (period) {
    case "7days":
      const sevenDaysAgo = new Date();
      sevenDaysAgo.setDate(now.getDate() - 7);
      filteredAuswertung = auswertung.value.filter((item) => {
        const itemDate = new Date(item.ts * 1000);
        return itemDate >= sevenDaysAgo && itemDate <= now;
      });
      break;
    case "1month":
      const oneMonthAgo = new Date();
      oneMonthAgo.setMonth(now.getMonth() - 1);
      filteredAuswertung = auswertung.value.filter((item) => {
        const itemDate = new Date(item.ts * 1000);
        return itemDate >= oneMonthAgo && itemDate <= now;
      });
      break;
    case "3months":
      const threeMonthsAgo = new Date();
      threeMonthsAgo.setMonth(now.getMonth() - 3);
      filteredAuswertung = auswertung.value.filter((item) => {
        const itemDate = new Date(item.ts * 1000);
        return itemDate >= threeMonthsAgo && itemDate <= now;
      });
      break;
    case "all":
      filteredAuswertung = auswertung.value;
      break;
  }

  console.log("Filtered Battery Data:", filteredAuswertung); // Debugging

  const labels = filteredAuswertung.map((item) => new Date(item.ts * 1000));
  const batteryData = filteredAuswertung.map((item) => item.battery);

  // Preisdaten nach Stunden truncieren
  const priceFilteredData = price.value.filter((item) => {
    const itemDate = new Date(item.Timestamp);
    return labels.some((label) => label.getTime() === itemDate.getTime());
  });

  console.log("Filtered Price Data:", priceFilteredData); // Debugging

  chartData.value.labels = labels;
  chartData.value.datasets[0].data = batteryData;
  chartData.value.datasets[1].data = priceFilteredData.map(
    (item) => item.Value
  );

  if (chart.value) {
    chart.value.update();
  } else {
    createChart();
  }
};

onMounted(() => {
  createChart();
});
</script>
