<script setup>
import { ref} from 'vue';

const APIKEYLOCATION = '2699ee3f0176484a97d6df5ac4f294d5';
const locationData = ref(null);
const APIKEYWEATHER = '6187e0f73b1e42b5a6870607232212';
const weatherData = ref(null);
let nextWeatherObject = (null);  

const getLocation = async () => {
    try {
        const response = await fetch(`https://ipgeolocation.abstractapi.com/v1/?api_key=${APIKEYLOCATION}`);
        const data = await response.json();
        locationData.value = data.city;
        
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}

const getWeather = async () => {
    try {
        if (!locationData.value) {
            await getLocation(); 
        }

        const response = await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${APIKEYWEATHER}&q=${locationData.value}&aqi=no&days=2&lang=en`);
        const data = await response.json();
        weatherData.value = data;
    

        await getNextWeather(weatherData.value); 
    } catch (error) {
        console.error('Error fetching data:', error);
    }
};

const getNextWeather = async (weatherData) => {
    try {
        if (!weatherData) {
            return;
        }

        const { localtime } = weatherData.location;
        

        let time = Number(localtime.slice(-5).slice(0, 2));
        const today = weatherData.forecast.forecastday[0].hour;
        const nextDay = weatherData.forecast.forecastday[1].hour;

        let current = time + 2 > 23 ? time + 2 - 24 : time + 2;
        let currentData = time + 2 > 23 ? nextDay : today;

        nextWeatherObject=[];


        for (let i = 0; i < 8; i++) {
            nextWeatherObject.push(currentData[current])
            
            if (current + 2 > 23) {
                currentData = nextDay;
            }
            current = current + 2 > 23 ? current + 2 - 24 : current + 2;
        }
        
        
    } catch (error) {
        console.error('Error getting next weather:', error);
    }
};

const handleClick = async () => {
    await getWeather(); 
};


</script>

<template>
  <div class="main-page">
   
    <h1>Weather App</h1>
   
    <v-btn @click="handleClick">Get Weather</v-btn>

    
    
    <div class="flex-container-current-weather-data">
        <div class="flex-item-current-weather-data" v-if="weatherData">
            <h2>Weather in {{ weatherData.location.name }}</h2>
            <p>{{ weatherData.current.temp_c }}°C</p>
            <p>{{ weatherData.current.condition.text }}</p>
            <p>Real Feel: {{ weatherData.current.feelslike_c }}°C</p>
    </div>

    </div>
    
    <div class="flexbox-container-next-weather-data">
      <div class="flexbox-items-next-weather-data" v-for="(nextWeather, index) in nextWeatherObject" :key="index">
        <h5>{{nextWeather.time.slice(-5)}}</h5>
        <img :src="nextWeather.condition.icon" :alt="nextWeather.condition.text" />
        <p>{{ nextWeather.temp_c }}°C</p>
      </div>
    </div>
  </div>
</template>

<style scoped>

.v-btn{
    margin-top: 1.5rem;
}

.main-page{
    font-family: "Reddit Mono", monospace;
    text-align: center;
    margin: 2rem;

  
}

.flexbox-container-next-weather-data
 {
    
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
    margin: 5rem;
}


.flex-container-current-weather-data{
    display:flex;
    justify-content: center;
    margin-top: 5rem;
    

}


</style>
