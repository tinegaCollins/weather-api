
<template>
<section>
  <p class="response" ref="responseParagraph">{{responseMessage}}</p>
  <nav>
    <img src="./assets/cytonn_logo.svg" alt="" srcset="">
  </nav>
  <div class="cordinates" v-if="ifDataFetched">
    <input type="number" step="0.0001" placeholder="latitude" v-model="latitude">
    <input type="number" step="0.0001" placeholder="longitude" v-model="longitude">
    <button @click="getData">get forecast</button>
  </div>
  <div class="data-in-charts" v-else>
    <button @click="back">change coordinates</button>
    <Line 
      :chart-data="temperaturesData" 
      :chart-options="temperaturesOptions" 
      css-classes="temperature-container"
    />
    <Line 
      :chart-data="humidityData" 
      :chart-options="humidityOptions" 
      css-classes="humidity-container"
    />
    <Line
      :chart-data="cloudCoverData"
      :chart-options="cloudCoverOptions"
      css-classes="cloud-cover-container"
    />
    <Line 
      :chart-data="windSpeedData"
      :chart-options="windSpeedOptions"
      css-classes="wind-speed-container"
    />
  </div>
</section>
</template>


<script setup>
import { ref, computed } from 'vue';
import { Line } from 'vue-chartjs'
import { Chart, LineController, PointElement, LineElement, CategoryScale , LinearScale} from 'chart.js';
Chart.register(LineController,LineElement, PointElement, CategoryScale, LinearScale);



const latitude = ref('');
const longitude = ref('');
const responseMessage = ref();
const responseParagraph = ref();

const displayResponce = (value)=>{
  responseMessage.value = value;
  responseParagraph.value.classList.add('slide-down');
  setTimeout(() => {
    responseMessage.value = undefined
    responseParagraph.value.classList.remove('slide-down')
  }, 3000);
}

const sixAmToSixPm = (value)=>{
  value.splice(0,6);
  value.splice(13, value.length);
  return value
}

let temperatures;
let humidity;
let cloudcover;
let windspeed;
let response;
const getData = async ()=>{
  if(latitude.value != '' && longitude.value != ''){
      try {
        const response = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude.value}&longitude=${longitude.value}&hourly=temperature_2m,relativehumidity_2m,cloudcover_mid,windspeed_120m`);
        const data = await response.json();
        temperatures = data.hourly.temperature_2m;
        humidity = data.hourly.relativehumidity_2m;
        cloudcover = data.hourly.cloudcover_mid;
        windspeed = data.hourly.windspeed_120m;
        sixAmToSixPm(temperatures);
        sixAmToSixPm(humidity);
        sixAmToSixPm(cloudcover);
        sixAmToSixPm(windspeed);
        ifDataFetched.value = false;
      }
      catch {
        if(latitude.value > 90 || latitude.value < -90){
          response = `Error: Latitude must be in range of -90 to 90°. Given:${latitude.value}°`
          displayResponce(response);
        }else if ( longitude.value > 179 || longitude.value < -180){
          response = `Error: Longitude must be in range of -180 to 180°. Given:${longitude.value}°`
          displayResponce(response);
        }else {
          response = "an error occurred, couldn't get the forecast";
          displayResponce(response);
        }
      }
  }else if(latitude.value == undefined || latitude.value == '' ){
    response = "enter a latitude";
    displayResponce(response);
  }
  else if ( longitude.value == undefined || longitude.value == ''){
    response = "enter a longitude";
    displayResponce(response);
  }
}
const ifDataFetched = ref(true);
const defaultLabels = ["6am","7am","8am","9am","10am","11am","12pm","1pm","2pm","3pm","4pm","5pm","6pm"];
const temperaturesData = computed(()=> ({
  labels : defaultLabels,
  datasets: [
    {
      label: 'Temperature (°C)',
      backgroundColor: '#ea5545',
      data: temperatures,
      borderColor: '#ea5545',
      fill: true
    }
  ]
}))

const temperaturesOptions = ref({
  plugins: {
    title: {
      text: 'Temperatures',
    }
  },
})

const humidityData = computed(()=> ({
  labels : defaultLabels,
  datasets: [
    {
      label: 'Humidity (%)',
      backgroundColor: '#87bc45',
      data: humidity,
      borderColor: "#87bc45"
    }
  ]
}))
const humidityOptions = ref({
  plugins: {
    title: {
      text: 'Humidity',
    }
  },
})
const cloudCoverData = computed(()=> ({
  labels : defaultLabels,
  datasets: [
    {
      label: 'Cloud Cover',
      backgroundColor: '#f46a9b',
      data: cloudcover,
      borderColor: '#f46a9b'
    }
  ]
}))
const cloudCoverOptions = ref({
  plugins: {
    title: {
      text: 'Cloud Cover',
    }
  },
})
const windSpeedData = computed(()=> ({
  labels : defaultLabels,
  datasets: [
    {
      label: 'Wind Speed (Km/h)',
      backgroundColor: '#3BADEF',
      data: windspeed,
      borderColor: '#3BADEF'
    }
  ]
}))
const windSpeedOptions = ref({
  plugins: {
    title: {
      text: 'Wind Speed',
    }
  },
})
const back = ()=>{
 window.location.reload();
}
</script>


<style>
*{
  margin: 0;
  padding: 0;
}
:root {
  --cytonn-green: #33887C;
}
body {
  font-family: "Open Sans", sans-serif;
}
nav {
  height: 7vh;
}
section {
  display: grid;
  place-items: center;
}
nav img {
  height: 80%;
  width: auto;
  margin-top: 10px;
}
section > p {
  position: absolute;
  top: 0;
  transition: transform 150ms cubic-bezier(0.39, 0.575, 0.565, 1);
  background-color: green;
  text-align: center;
  border-radius: 5px;
  width: 80%;
}
.slide-down{
  padding: 5px 10px;
  transform: translateY(20vh);
}
.cordinates {
  height: calc(93vh - 50px);
  margin-top: 35px;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  padding: 10px;
  row-gap: 20px;
  align-items: center;
  flex-direction: column;
}
.cordinates > * {
  position: relative;
  bottom: 50px;
  right: 10px;
}
.cordinates input {
  width: 100%;
  height: 40px;
  outline: none;
  border: 1px solid black;
  border-radius: 5px;
  padding: 0 10px;
}
.cordinates button {
  font-size: 1.1rem;
  outline: none;
  appearance: none;
  background-color: var(--cytonn-green);
  border: 1px solid black;
  padding: 7px 15px;
  border-radius: 5px;
}
.cordinates button:hover {
  cursor: pointer;
}
.data-in-charts {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  padding: 15px 15px 30px 15px;
  height: 60vh;
  position: relative;
}
.data-in-charts > * {
  width: 80vw;
  height: auto;
}
.data-in-charts .temperature-container {
  margin-top: 20px;
}
.data-in-charts button {
  position: absolute;
  top: 0;
  background-color: var(--cytonn-green);
  border: 1px solid black;
  padding: 7px 15px;
  border-radius: 30px;
  font-size: 1.1rem;
  outline: none;
  appearance: none;
  cursor: pointer;
  width: max-content;
}
@media screen and (min-width: 600px) {
  section > p {
    width: max-content;
  }
  .data-in-charts > * {
    width: 300px;
    /* margin-top: 30px; */
  }
}
@media screen and (min-width: 1024px) {
  .cordinates {
    flex-direction: row;
    column-gap: 30px;
  }
  .cordinates input {
    width: max-content;
  }
  .data-in-charts {
    justify-content: space-around;
  }
  .data-in-charts > * {
    width: 470px;
    height: auto;
  }
  .temperature-container , .humidity-container {
    margin-top: 30px;
  }
}

</style>
