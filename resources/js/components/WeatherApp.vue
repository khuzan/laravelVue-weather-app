<template>
    <div id="mainapp" class="text-white mb-8" 
        v-bind:class="{ 
        warm: Math.round(TempConversion(currentTemperature.actual)) > 30,
        cold: Math.round(TempConversion(currentTemperature.actual)) <= 0
    }">
        <main>
            <div class="search-box">
                <input type="text" class="search-bar" placeholder="Search place...." v-model="query" 
                @keyup.enter="fetchWeather">
            </div>

            <div v-if="loading" class="loading">
                <lottie-player src="https://assets1.lottiefiles.com/private_files/lf30_jmgekfqg.json"  background="transparent"  speed="1"  style="width: 300px; height: 300px;"  loop autoplay></lottie-player>
            </div>

            <div v-else class="weather-wrap">
                <div class="location-box">
                    <div class="location">{{ location.city }}</div>
                    <div class="date">{{ toDayDate(location.date) }}</div>
                </div>
                
                <div class="weather-box">
                    <div class="flex items-center weather-box">
                        <div>
                            <div class="temp">{{ Math.round(TempConversion(currentTemperature.actual)) }}°C</div>
                        </div>
                        <div class="mx-5">
                            <div class="weather-icon">
                                <img :src="'https://openweathermap.org/img/wn/'+ currentTemperature.icon + '@2x.png'">
                            </div>
                            <div class="feels">Feels like {{ Math.round(TempConversion(currentTemperature.feels)) }}°C</div>
                        </div>
                    </div>

                    <div class="weather">{{ currentTemperature.summary}}</div>
                    <div class="weather-description">{{ currentTemperature.description}}</div>
                </div>
                
                <div class="future-weather text-sm font-semibold mt-8 px-6 py-8 overflow-hidden">
                    <div 
                        v-for="(day, index) in daily"
                        :key="day.dt"
                        class="flex items-center"
                        :class="{ 'mt-8' : index > 0 }"
                        >
                        <div class="w-2/6 text-lg text-gray-200">{{ toDayOfWeek(day.dt) }}</div>
                        <div class="w-4/6 px-4 flex items-center">
                            <div><img :src="'https://openweathermap.org/img/wn/'+ day.weather[0].icon + '@2x.png'" width="30" height="30"></div>
                            <div class="ml-3">{{ day.weather[0].description }}</div>
                        </div>
                        <div class="w-1/6 text-right">
                            <div>{{ Math.round(TempConversion(day.temp.max)) }}°C</div>
                            <div>{{ Math.round(TempConversion(day.temp.min)) }}°C</div>
                        </div>
                    </div>
                </div>
            </div>
        </main>
    </div>
</template>

<script>
    export default {
        mounted() {
            this.loading = true;
            this.fetchData()
        },
        data() {
            return {
                loading: true,
                apiKey: '39a8ff16787f42488cdf93ab214cfb92',
                query: '',
                currentTemperature: {
                    actual: '',
                    feels: '',
                    summary: '',
                    icon: '',
                },
                daily: [],
                location: {
                    city: 'America/Chicago',
                    address: '',
                    lat: 33.44,
                    lng: -94.04,
                    date: 'Monday 20 January 2020',
                },
            }
        },
        watch : {
            location: {
                handler(){
                    this.fetchData()
                },
                deep: true
            }
        },
        methods: {
            fetchData() {
                fetch(`api/weather?lat=${this.location.lat}&lng=${this.location.lng}`)
                .then(response => response.json())
                .then(data => {
                    this.loading = false;
                    console.log(data);
                    this.currentTemperature.actual = Math.round(data.current.temp)
                    this.currentTemperature.feels = Math.round(data.current.feels_like)
                    this.currentTemperature.summary = data.current.weather[0].main
                    this.currentTemperature.description = data.current.weather[0].description
                    this.currentTemperature.icon = data.current.weather[0].icon
                    this.location.date = data.daily[0].dt
                    
                    this.daily = data.daily
                })
            },
            toDayOfWeek(timestamp) {
                const newDate = new Date(timestamp*1000)
                const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']

                return days[newDate.getDay()]
            },
            toDayDate(timestamp) {
                const newDate = new Date(timestamp * 1000);
                const months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
                const toDate = newDate.getDate() + ' ' + months[newDate.getMonth()] + ' ' + newDate.getFullYear();

                return toDate;
            },
            TempConversion(kelvin){
                const celcius = kelvin - 273.15

                return celcius;
            },
            fetchWeather() {
                this.loading = true;
                fetch(`api/places?query=${ this.query }`)
                .then(res => {
                    return res.json();
                }).then(this.setResults);
            },
            setResults(results) {
                this.location.city = results.features[0].properties.formatted;
                this.location.lat  = results.features[0].properties.lat;
                this.location.lng  = results.features[0].properties.lon;
            }
        }
    }

    
    
</script>
