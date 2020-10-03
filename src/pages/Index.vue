<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-md q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearchErrorHandle"
        placeholder="Search"
        dark
        borderless
        ref="searchLocation"
        :error="[val => !!val || 'Field is required']"
      >
        <template v-slot:before>
          <q-icon @click="getWeatherByCoords" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getLocation" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C </span>
        </div>

        <div id="info">
          <i class="fas fa-sun"></i> {{ sunrise }} <i class="far fa-sun"></i>
          {{ sunset }} <i class="fas fa-tint"></i> {{ humidity }}
          <i class="far fa-sun"></i> {{ pressure }} <i class="fas fa-wind"></i>
          {{ wind }}
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="
            `http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`
          "
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar <br />
          Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      minTemp: "",
      maxTemp: "",
      sunrise: "",
      sunset: "",
      pressure: "",
      humidity: "",
      wind: "",
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "83c8782a5f9b96efe9de7559c9deb360"
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show();
      navigator.geolocation.getCurrentPosition(position => {
        console.log("position", position);
        this.lat = position.coords.latitude;
        this.lon = position.coords.longitude;
        this.getWeatherByCoords();
      });
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
      ).then(response => {
        this.pressure = response.data.main.pressure;
        this.humidity = response.data.main.humidity + "%";
        this.wind = response.data.wind.speed + "m/s";
        this.overcast = response.data.weather[0].description;
        this.icon =
          "images/" + response.data.weather[0].icon.slice(0, 2) + ".svg";
        this.sunrise = new Date(
          response.data.sys.sunrise * 1000
        ).toLocaleTimeString("en-PH", {
          hour: "2-digit",
          minute: "2-digit",
          hour12: true
        });
        this.sunset = new Date(
          response.data.sys.sunset * 1000
        ).toLocaleTimeString("en-PH", {
          hour: "2-digit",
          minute: "2-digit",
          hour12: true
        });
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },

    getWeatherBySearchErrorHandle() {
      this.$refs.searchLocation.validate();
      if (!this.$refs.searchLocation.hasError) {
        this.getWeatherBySearch();
      }
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`
      )
        .then(response => {
          this.pressure = response.data.main.pressure;
          this.humidity = response.data.main.humidity + "%";
          this.wind = response.data.wind.speed + "m/s";
          this.overcast = response.data.weather[0].description;
          this.icon =
            "images/" + response.data.weather[0].icon.slice(0, 2) + ".svg";
          this.sunrise = new Date(
            response.data.sys.sunrise * 1000
          ).toLocaleTimeString("en-PH", {
            hour: "2-digit",
            minute: "2-digit",
            hour12: true
          });
          this.sunset = new Date(
            response.data.sys.sunset * 1000
          ).toLocaleTimeString("en-PH", {
            hour: "2-digit",
            minute: "2-digit",
            hour12: true
          });
          this.weatherData = response.data;
          this.$q.loading.hide();
        })
        .catch(error => {
          if (error.message) {
            this.$q.dialog({
              dark: true,
              title: "Alert",
              message: "Location Not Found"
            });
            this.$q.loading.hide();
          }
        });
    }
  }
};
</script>

<style lang="stylus">
.q-page{
  background: linear-gradient(to bottom, #136a8a, #267871);
  &.bg-night{
    background: linear-gradient(to bottom, #232526, #414345);
  }
  &.bg-day{
    background: linear-gradient(to right, #00b4db, #0083b0);
  }
}


.degree{
  top: -44px;
}

.skyline{
  flex: 0 0 150px;
  background: url(../../public/skyline.png)
  background-size: contain;
  background-position: center bottom
}

.q-field__messages{
  font-size: 15px
}

.icon {
  position: inherit;
  top: 2px;
  padding-left: 8px;
}

#temp-values {
  text-align: right;
  text-justify: distribute;
  display: block;
  position: relative;
  top: -60px;
}
</style>
