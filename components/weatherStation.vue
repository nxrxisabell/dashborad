<template>
    <div v-if="weatherStation">
     <p>battery: {{weatherStation.battery}}%
        <div v-for="(value, key) in weatherStation.collects" :key="key">
          <template v-if="key.toString().includes(`ut`) === false">
            {{ weatherStation.collectSettings[key].dataType }}: {{ value }} {{ weatherStation.collectSettings[key].unit }}
          </template>
        </div>
      </p>
    </div>
  
  </template>
  <script setup lang=ts>
  const props = defineProps(['auth'])

  const  {data}  = await useFetch("https://eu.iot-api.milesight-iot.com/v1/devices", {method: "get", headers: {Authorization:`Bearer ${props.auth.jws}`, tokenid: props.auth.tokenid
   }})
  const weatherStation = data.value.data[0]
  console.log(weatherStation)
  
  </script>