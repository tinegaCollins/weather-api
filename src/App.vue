
<template>
<section>
  <nav>
    <img src="./assets/cytonn_logo.svg" alt="" srcset="">
  </nav>
  <div class="cordinates">
    <input type="number" step="0.0001" placeholder="latitude" v-model="latitude">
    <input type="number" step="0.0001" placeholder="longitude" v-model="longitude">
    <button @click="getData">get forecast</button>

  </div>
</section>
</template>


<script setup>
import { ref } from 'vue';
const latitude = ref();
const longitude = ref();
const temperatureData = ref();
const test = [1,2,3,4,5,6,7,8,9,9,9]
const sixAmToSixPm = (data)=>{
  data.splice(0,5);
  data.splice(11,data.length + 1)
  return data
}

console.log(sixAmToSixPm(test));


const getData = async ()=>{
  console.log(latitude.value,longitude.value);
  if(latitude.value != null && latitude.value != '' || 
    longitude.value != null && longitude.value != ''){
      const response = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude.value}&longitude=${longitude.value}&hourly=temperature_2m,relativehumidity_2m,cloudcover_mid,windspeed_120m`);
      const data = await response.json();
      const temperatures = data.hourly.temperature_2m;
  }
  else {
    console.log("bot ");
  }

}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
nav {
  height: 7vh;
}
nav img {
  height: 80%;
  width: auto;
  margin-top: 10px;
}
</style>
