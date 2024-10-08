<template>
    {{ price.data }}
</template>
<script setup lang="ts">
const price = await useFetch("https://www.fingrid.fi/api/graph/dataset?configurationItems%5B0%5D.variableId=244&configurationItems%5B1%5D.variableId=106&start=2024-09-26+00%3A00%3A00&end=2024-10-13+23%3A59%3A59")

// const truncateToHour = (date) => {
//   const newDate = new Date(date);
//   newDate.setMinutes(0, 0, 0); // Setzt Minuten, Sekunden und Millisekunden auf 0
//   return newDate.getTime(); // Gibt den Zeitstempel in Millisekunden zurück
// };
// const labels = filteredData.map(item => new Date(item.ts * 1000)); // Setze die richtigen Zeitstempel für Labels
// const labelsTruncated = labels.map(label => truncateToHour(label))
// const batteryData = filteredData.map(item => item.battery);
// const filteredData = auswertung.filter((item, index, arr) => {
//   return index === 0 || item.battery !== arr[index - 1].battery;
// });

// const priceDataTruncated = priceData.map(item => ({
//   ts: truncateToHour(new Date(item.Timestamp)),
//   value: item.Value,
// }));

// const priceFilteredData = priceDataTruncated.filter(item =>
//   labelsTruncated.includes(item.ts)
// );


// // Extrahiere die Werte für die Preis-Daten
// const filteredPriceData = priceFilteredData.map(item => item.value);

// // Überprüfe die Längen
// if (labels.length !== filteredPriceData.length) {
//   console.log(labels.length, filteredPriceData.length)
//   console.warn('Die Länge der Labels und der Preis-Daten stimmt nicht überein.');
// }
    
// //     const filteredData = auswertung.filter((item, index, arr) => {
// //         return index === 0 || item.battery !== arr[index -1].battery
// //     })
// //     const   labels = filteredData.map(item => new Date(item.ts * 1000)) 
// //     const    batteryData = filteredData.map(item => item.battery)
// //     const priceData = price.data.value
// //   .filter(item => labels.some(label => new Date(item.ts * 1000).getTime() === label.getTime()))
// //   .map(item => item.value);

// //     if (labels.length !== priceData.length) {
// //   console.warn('Die Länge der Labels und der Preis-Daten stimmt nicht überein.');
// // }
   
</script>