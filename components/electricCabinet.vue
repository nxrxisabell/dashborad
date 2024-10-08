<template>
    <div v-if="electricCabinet">
     <p>
        <div v-for="(value, key) in electricCabinet.collects" :key="key">
          <template v-if="key.toString().includes(`ut`) === false">
            {{ electricCabinet.collectSettings[key].dataType }}: {{ value }} {{ electricCabinet.collectSettings[key].unit }}
          </template>
        </div>
      </p>
    </div>
  
  </template>
  <script setup lang=ts>
  const props = defineProps(['auth'])

  const  {data}  = await useFetch("https://eu.iot-api.milesight-iot.com/v1/devices", {method: "get", headers: {Authorization:`Bearer ${props.auth.jws}`, tokenid: props.auth.tokenid
   }})
  const electricCabinet = data.value.data[1]
  console.log(electricCabinet)
  
  </script>