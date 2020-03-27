<template>
  <div id="weather_main">
    <div id="successCheck">
      <div>
        <svg class="checkmark" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 52 52"><circle class="checkmark__circle" cx="26" cy="26" r="25" fill="none"/><path class="checkmark__check" fill="none" d="M14.1 27.2l7.1 7.2 16.7-16.8"/></svg>
      </div>
    </div>

    <main>
      <div class="search_box">
        <button class="header_button clear_button" @click="clearSearch">
          <i class="fas fa-chevron-left"></i>
        </button>
        <button
          v-if="typeof weather_search_res.main != 'undefined'"
          @click="setFavoritePlace"
          class="header_button favorite_button"
          title="Like"
        ><i class="far fa-heart"></i></button>
        <input
          id="search_input"
          type="text"
          class="search_bar"
          placeholder="Search ..."
          v-model="query"
          @keypress="getWeather"
        />
        <button class="search_button" @click="getWeather" title="Search">Search</button>
      </div>

      <div class="search_weather_wrap" v-if="typeof weather_search_res.main != 'undefined'">
        <div
          class="search_error"
          v-if="weather_search_res.main == 'error'"
        >! {{ weather_search_res.message }} !</div>
        <div class="location_box" v-if="weather_search_res.name">
          <div class="location">
            <p>
              {{ weather_search_res.name }},
              {{ weather_search_res.sys.country }}
            </p>
          </div>
        </div>

        <div class="weather_box" v-if="weather_search_res.weather">
          <div class="temp">
            <p>{{ Math.round(weather_search_res.main.temp) }} °C</p>
          </div>
          <div class="weather">
            <p>{{ weather_search_res.weather[0].main }}</p>
          </div>

          <div class="weather_img">
            <img v-bind:src="weather_img" />
          </div>
        </div>
      </div>

      <div class="default_weather_wrap" v-if="weather_default_res">
        <table>
          <tr v-for="weather in weather_default_res" :key="weather.name">
            <td class="default_loc_name">
              <p>{{ weather.name }}</p>
            </td>
            <td class="default_loc_temp">
              <p>{{ Math.round(weather.main.temp) }} °C</p>
            </td>
            <td class="default_loc_weather">
              <p>{{ weather.weather[0].main }}</p>
            </td>
          </tr>
        </table>
      </div>

      <footer>
        <div class="date">
          <p>{{ date() }}</p>
        </div>
      </footer>
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      api_key: "d2a9887d08726ec158f84a1205d0b8f3",
      url_base: "https://api.openweathermap.org/data/2.5/",
      query: "",
      weather_img: "",
      weather_default_ids: {
        id_1: "3067696",
        id_2: "3078610",
        id_3: "3068799"
      },
      weather_search_res: {},
      weather_default_res: {}
    };
  },
  methods: {
    /**
     * Get weather based on user search or default
     */
    getWeather(e) {
      if ((e.key == "Enter" || e.type == "click") && this.query != "") {
        this.getSearchWeather();
      } else if (e == "default") {
        this.getDefaultWeather();
      }
    },

    /**
     * Search for user selected weather
     */
    getSearchWeather() {
      fetch(
        `${this.url_base}weather?q=${this.query}&&units=metric&appid=${this.api_key}`
      )
        .then(res => {
          return res.json();
        })
        .then(this.setSearchResults);
    },

    /**
     * Search for default weather
     */
    getDefaultWeather() {
      fetch(
        `${this.url_base}group?id=${this.weather_default_ids.id_1},${this.weather_default_ids.id_2},${this.weather_default_ids.id_3}&units=metric&appid=${this.api_key}`
      )
        .then(res => {
          return res.json();
        })
        .then(this.setDefault);
    },

    /**
     * Set the search result
     */
    setSearchResults(results) {
      if (results.cod == "400" || results.cod == "404") {
        this.weather_search_res = {};
        this.weather_default_res = {};
        this.weather_search_res.message = "City not found";
        this.weather_search_res.main = "error";
      } else {
        this.weather_search_res = results;
        this.city_id = results.id;
        this.weather_default_res = null;
        let img_src;
        switch (results.weather[0].main) {
          case "Rain":
            img_src = "rain";
            break;
          case "Clouds":
            img_src = "clouds";
            break;
          case "Clear":
            img_src = "clear_sky";
            break;
          case "Thunderstorm":
            img_src = "thunderstorm";
            break;
          case "Snow":
            img_src = "snow";
            break;
          case "Drizzle":
            img_src = "drizzle";
            break;
        }
        this.weather_img = "weather-img/" + img_src + ".png";
      }
    },

    /**
     * Set default cities
     */
    setDefault(results) {
      this.weather_default_res = results.list;
      this.weather_search_res = {};
    },

    /**
     * Get current date
     */
    date() {
      var today = new Date();
      var dd = String(today.getDate()).padStart(2, "0");
      var mm = String(today.getMonth() + 1).padStart(2, "0");
      var yyyy = today.getFullYear();
      today = dd + "." + mm + "." + yyyy;
      return today;
    },

    /**
     * Clear the search result
     */
    clearSearch() {
      this.getWeather("default");
    }
  },
  beforeMount() {
    this.getWeather("default");
  }
};
</script>

