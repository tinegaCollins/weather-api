
<template>
<section>
  <p class="response" ref="responseParagraph">{{responseMessage}}</p>
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
        console.log('done');
      }
      catch {
        if(latitude.value > 90 || latitude.value < -90){
          response = `Error: Latitude must be in range of -90 to 90°. Given:${latitude.value}°`
          displayResponce(response);
        }else if ( longitude.value > 180 || longitude.value < -180){
          response = `Error: Longitude must be in range of -180 to 180°. Given:${longitude.value}°`
          displayResponce(response);
        }else {
          response = 'an error occurred, could`t get the forecast'
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
  margin-left: 35vw;
  border-radius: 5px;
}
.slide-down{
  padding: 5px 10px;
  transform: translateY(20vh);
}
.cordinates {
  height: calc(93vh - 45px);
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
}
.cordinates input {
  width: 70%;
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
@media screen and (min-width: 600px) {
  section > p{
    margin-left: 42vw;
  }
}
@media screen and (min-width: 1024px) {
  section > p {
    margin-left: 45vw;
  }
  .cordinates {
    flex-direction: row;
    column-gap: 30px;
  }
  .cordinates input {
    width: max-content;
  }
}
</style>
