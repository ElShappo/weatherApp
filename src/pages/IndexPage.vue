<template>
  <q-page class="flex column" style="height: 100vh" :class="bgClass">

    <div class="col-3 q-pt-lg q-px-lg">
      <q-input bottom-slots v-model="search" placeholder="Search" dark error-message="Type something" borderless @keyup.enter="this.getWeatherBySearch">

        <template v-slot:prepend>
          <q-icon name="my_location" @click="getLocation"/>
        </template>

        <template v-slot:append>
          <q-icon name="search" class="cursor-pointer" @click="this.getWeatherBySearch" />
        </template>

      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col-3 text-teal-1 text-center">
        <div class="text-h4 text-weight-light">
          {{this.weatherData.name}}
        </div>

        <div class="text-h7 text-weight-light">
          {{weatherData.weather[0].main}}
        </div>

        <div class="text-h2 text-weight-thin q-mt-xl">
          <span>{{Math.round(weatherData.main.temp)}}</span>
          <span class="temperature text-h5">&degC</span>
        </div>
      </div>

      <div class="col-1 text-center">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"/>
      </div>
    </template>

    <template v-else>
      <div class="col-2 column text-white text-weight-light text-center text-h4">
        Quasar Weather App
      </div>

      <q-btn flat @click="getLocation" class="col-2 column text-white">
        <q-icon left size="3em" name="my_location" />
        <div>Find my location</div>
      </q-btn>
    </template>


    <div class="col-5 skyline">
    </div>

  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import axios from 'axios'

export default {
  name: 'IndexPage',

  data() {
    return {
      search: '',
      weatherData: null,
      lat: '',
      lon: '',
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: 'fac305d12b617f22c4766ab39e1e8ac0',
    }
  },

  computed: {
    bgClass: function() {
      if (this.weatherData) { // check if we've chosen the place
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        }
        return 'bg-day'
      }
    }
  },

  methods: {
    getLocation: function() {
      const options = {
        enableHighAccuracy: true,
        timeout: 400,
        maximumAge: 0
      };

      function error(err) {
        console.warn(`ERROR(${err.code}): ${err.message}`);
      }

      if (this.$q.platform.is.electron) {
        axios.get(`http://ip-api.com/json/`).then(response => {
          console.log("Response: ", response)
          this.lat = response.data.lat
          this.lon = response.data.lon

        }).then(this.getWeatherByCoords)
        .catch(function() {console.warn("something went wrong during promise")})

      }
      else {
        console.log('this is not electron')
        navigator.geolocation.getCurrentPosition(pos => {
          const crd = pos.coords;

          this.lat = crd.latitude
          this.lon = crd.longitude

          console.log('Your current position is:');
          console.log(`Latitude : ${this.lat}`);
          console.log(`Longitude: ${this.lon}`);
          console.log(`More or less ${crd.accuracy} meters.`);

        }, error, options);
        // 1. "succcess", "error" are callback functions taking one parameter
        // 2. we are unable to determine beforehand what parameter it is exactly
        //    that will be passed to "success" and "error" callbacks within "getCurrentPosition"

        setTimeout(this.getWeatherByCoords, 500)
      }
    },

    getWeatherByCoords: function() {

      // console.log(`Latitude : ${this.lat}`);
      // console.log(`Longitude: ${this.lon}`);
      // check whether the timeout set previously was sufficient to receive the latitude and longitude

      this.$q.loading.show()

      axios.get(`${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`).then(response => {
        // query to weatherMapApi
        this.weatherData = response.data;
        this.$q.loading.hide()
        console.log(this.weatherData)
      }).catch( () => {
        console.warn('query answer was not received')
        var self = this
        self.$q.loading.hide()

        this.$q.notify({
          message: `Can't find the place`,
          color: 'negative'
        })
      })
    },

    getWeatherBySearch: function() {
      this.$q.loading.show()
      console.log("getWeatherBySearch activated")

      axios.get(`${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`).then(response => {
        // query to weatherMapApi

        this.weatherData = response.data;
        this.$q.loading.hide()
        console.log(this.weatherData)

      }).catch( () => {
        console.warn('no match for the query')
        var self = this
        self.$q.loading.hide()

        this.$q.notify({
          message: `Can't find "${this.search}"`,
          color: 'negative'
        })

      })
    }
  }
}
</script>

<style lang="sass">

  .q-page
    background: #43cea2
    background: -webkit-linear-gradient(to bottom, #185a9d, #43cea2) /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(to bottom, #185a9d, #43cea2) /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */

    &.bg-night
      background: #C04848  /* fallback for old browsers */
      background: -webkit-linear-gradient(to top, #480048, #C04848)  /* Chrome 10-25, Safari 5.1-6 */
      background: linear-gradient(to top, #480048, #C04848) /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */


    &.bg-day
      background: #4568DC  /* fallback for old browsers */
      background: -webkit-linear-gradient(to bottom, #B06AB3, #4568DC)  /* Chrome 10-25, Safari 5.1-6 */
      background: linear-gradient(to bottom, #B06AB3, #4568DC) /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */


  .temperature
    vertical-align: top

  .avatar
    width: 140px
    height: 100px

  .skyline
    background: no-repeat url(~assets/skyline.svg)
    background-size: 100%
    background-position: center bottom


</style>
