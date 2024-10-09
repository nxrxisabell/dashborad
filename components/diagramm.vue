<template>
  <div>
    <button @click="filterData('7days')">Last 7 Days</button>
    <button @click="filterData('1month')">Last Month</button>
    <button @click="filterData('3months')">Last 3 Month</button>
    <button @click="filterData('all')">All Data</button>
  </div>
  <div
    v-if="auswertung"
    style="max-width: 1300px !important; max-height: 300px; height: 300px"
  >
    <canvas id="myChart"></canvas>
  </div>
  <!-- <Line :data="chartData" :options="chartOptions" :update-mode="'default'" /> -->
</template>
<script setup lang="ts">
import { Line } from "vue-chartjs";
import {
  Chart as ChartJS,
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
  Chart,
} from "chart.js";
import "chartjs-adapter-dayjs";

ChartJS.register(
  CategoryScale,
  LinearScale,
  LineController,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  TimeScale
);
let chart;
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
const auswertung = data.value.data;
const date = new Date();
console.log(date.getFullYear(), "Month", date.getMonth() + 1, date.getDate());
const price = await useFetch(
  `https://www.fingrid.fi/api/graph/dataset?configurationItems%5B0%5D.variableId=244&configurationItems%5B1%5D.variableId=106&start=2024-09-26+00%3A00%3A00&end=${date.getFullYear()}-${
    date.getMonth() + 1
  }-${date.getDate()}+23%3A59%3A59`
);

let filteredData;
let labels;
let batteryData;
let filteredPriceData;
let labelsTruncated;

const chartData = ref({
  labels: [],
  datasets: [
    {
      label: "Battery",
      backgroundColor: "#f87979",
      data: [],
      yAxisID: "y",
    },
    {
      label: "Price",
      backgroundColor: "#g86786",
      data: [],
      yAxisID: "y1",
    },
  ],
});
const chartOptions = ref({
  responsive: true,
  maintainAspectRatio: false,

  scales: {
    x: {
      type: "time",
      time: {
        unit: "day",
      },
      title: {
        display: true,
        text: "Day",
      },
    },

    y: {
      title: {
        display: true,
        text: "Batterie (%)",
      },
      position: "left",
      min: 0,
      max: 100,
    },
    y1: {
      title: {
        display: true,
        text: "Price (€)",
      },
      position: "right",
    },
  },
});
onMounted(() => {
  const ctx = document.getElementById("myChart").getContext("2d");
  chart = new Chart(ctx, {
    type: "line",
    data: chartData.value,
    options: chartOptions.value,
  });
});

async function filterData(period) {
  const now = new Date();

  switch (period) {
    case "7days":
      const sevenDaysAgo = new Date(now);
      sevenDaysAgo.setDate(now.getDate() - 7);
      filteredData = auswertung.filter((item) => {
        const itemDate = new Date(item.ts * 1000);
        return itemDate >= sevenDaysAgo && itemDate <= now; // Filtere nur Daten in den letzten 7 Tagen
      });
      await fertigData();

      break;
    case "1month":
      const startOfPreviousMonth = new Date(
        now.getFullYear(),
        now.getMonth() - 1,
        1
      );
      const startOfCurrentMonth = new Date(
        now.getFullYear(),
        now.getMonth(),
        1
      );

      filteredData = auswertung.filter((item) => {
        const itemDate = new Date(item.ts * 1000);
        return (
          itemDate >= startOfPreviousMonth && itemDate < startOfCurrentMonth
        ); // Filtere nur Daten im letzten Monat
      });
      await fertigData();

      break;
    case "3months":
      const threeMonthsAgo = new Date(now);
      threeMonthsAgo.setMonth(now.getMonth() - 3);
      filteredData = auswertung.filter((item) => {
        const itemDate = new Date(item.ts * 1000);
        return itemDate >= threeMonthsAgo && itemDate <= now; // Filtere nur Daten in den letzten 3 Monaten
      });
      fertigData();

      break;
    case "all":
      filteredData = auswertung.filter((item, index, arr) => {
        return index === 0 || item.battery !== arr[index - 1].battery;
      });
      fertigData();
      break;
  }
}
await filterData("all");

async function fertigData() {
  labels = filteredData.map((item) => new Date(item.ts * 1000));

  batteryData = filteredData.map((item) => item.battery);

  const truncateToHour = (date) => {
    const newDate = new Date(date);
    newDate.setMinutes(0, 0, 0);
    return newDate.getTime();
  };

  labelsTruncated = labels.map((label) => truncateToHour(label));

  const priceDataTruncated = price.data.value[0].Values.map((item) => ({
    ts: truncateToHour(new Date(item.Timestamp)),
    value: item.Value,
  }));

  const priceFilteredData = priceDataTruncated.filter((item) =>
    labelsTruncated.includes(item.ts)
  );
  console.log(priceFilteredData);
  const filteredPriceData = labelsTruncated.map((ts) => {
    const priceItem = priceFilteredData.find((item) => item.ts === ts);
    return priceItem ? priceItem.value : null; // null, wenn kein passender Preis gefunden
  });
  console.log(filteredPriceData);
  chartData.value.datasets[0].data = batteryData;
  chartData.value.datasets[1].data = filteredPriceData;
  chartData.value.labels = labelsTruncated;
  chart.update();
}
</script>
