<template>
  <section class="w-full h-full">
    <div
      class="
        flex flex-col
        m-auto
        max-w-md
        min-h-screen
        p-1
        bg-gradient-to-r
        from-blue-900
        to-blue-800
      "
    >
      <div class="px-2 py-1 rounded-md" style="background-color: #e46b1b">
        <h1 class="text-lg text-white font-bold">Weather App</h1>
      </div>
      <div class="bg-blue-600 px-2 py-1 rounded-md mt-2">
        <h1 class="text-lg text-white font-light">
          Checking weather data for: {{ inputCity }}
        </h1>
      </div>
      <div class="flex flex-row justify-between mt-2">
        <input
          v-model="inputCity"
          class="
            w-8/12
            border
            bg-gray-100
            px-2
            py-1
            text-light
            rounded-l-md
            focus:outline-none
          "
          :placeholder="inputCity ? '' : 'Write city'"
        />
        <button
          @click="getWeather"
          class="
            bg-yellow-600
            hover:bg-yellow-700
            text-white
            font-bold
            py-1
            px-4
            rounded-r-md
            text-sm
            w-4/12
            focus:outline-none
          "
        >
          Get weather!
        </button>
      </div>
      <div
        v-if="cityFound"
        class="flex flex-col m-auto space-y-4 min-w-full weather-animation"
      >
        <div class="border rounded-2xl shadow-xl px-10 py-10 mx-3 bg-white">
          <h1
            v-if="weatherData.city && weatherData.country"
            class="text-3xl font-bold text-center"
          >
            {{ weatherData.city }}, {{ weatherData.country }}
          </h1>
          <div class="flex flex-row space-x-2 mt-2 justify-center">
            <img
              v-if="weatherData.iconUrl"
              :src="weatherData.iconUrl"
              width="70"
              height="70"
            />
            <h1 v-if="weatherData.temperature" class="text-4xl font-bold">
              {{ weatherData.temperature }}<span>&#8451;</span>
            </h1>
          </div>
        </div>
        <div class="border rounded-2xl shadow-xl px-10 py-10 mx-3 bg-white">
          <div class="font-light">
            <h1 v-if="weatherData.temperatureFellsLike">
              Feels like: {{ weatherData.temperatureFellsLike
              }}<span>&#8451;</span>
            </h1>
            <h1 v-if="weatherData.temperatureMin">
              Temp min: {{ weatherData.temperatureMin }}<span>&#8451;</span>
            </h1>
            <h1 v-if="weatherData.temperatureMax">
              Temp max: {{ weatherData.temperatureMax }}<span>&#8451;</span>
            </h1>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { ref } from "vue";
import axios from "axios";

export default {
  setup() {
    const weatherApiKey = "068578c46d8d37243d746e79790a94ac";
    const inputCity = ref("");
    const cityFound = ref(false);

    const weatherData = ref({
      city: "",
      country: "",
      temperature: "",
      temperatureFellsLike: "",
      temperatureMin: "",
      temperatureMax: "",
      iconUrl: "",
    });

    const getWeather = () => {
      axios
        .get(
          `https://api.openweathermap.org/data/2.5/weather?q=${inputCity.value}&appid=${weatherApiKey}&units=metric`
        )
        .then((response) => {
          console.log(response.data);
          cityFound.value = false;

          const {
            data: { main, name, weather, sys },
          } = response;

          const ref = weatherData.value;

          ref.city = name;
          ref.country = sys.country;
          ref.temperature = parseInt(main.temp);
          ref.temperatureFellsLike = parseInt(main.feels_like);
          ref.temperatureMin = main.temp_min;
          ref.temperatureMax = main.temp_max;
          ref.iconUrl = `https://openweathermap.org/img/w/${weather[0].icon}.png`;
        })
        .finally(() => {
          inputCity.value = "";
          cityFound.value = true;
        });
    };

    return {
      cityFound,
      inputCity,
      getWeather,
      weatherData,
    };
  },
};
</script>

<style>
.weather-animation {
  animation-name: show_weather_data;
  animation-duration: 0.5s;
  animation-timing-function: ease-out;
}

@keyframes show_weather_data {
  from {
    opacity: 0;
    transform: translate(-100%, 0);
  }

  to {
    opacity: 1;
    transform: translate(0, 0);
  }
}
</style>