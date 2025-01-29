<template>
  <section
    class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-r from-blue-500 to-indigo-600 text-white">
    <h1 class="text-4xl font-bold mb-8">Weather App</h1>
    <div class="flex flex-col">
      <!-- Formulario -->
      <form @submit.prevent="getWeather"
        class="flex flex-row max-w-md items-center gap-6 bg-white text-gray-800 p-6 rounded-lg shadow-lg">
        <input type="text" v-model="location" placeholder="Enter location"
          class="w-96 px-4 py-2 border rounded-md focus:outline-none focus:ring focus:ring-blue-500" required />
        <button type="submit"
          class="bg-blue-600 text-white text-sm font-bold px-4 py-2 rounded-md hover:bg-blue-700 focus:outline-none focus:ring focus:ring-blue-500">
          Get Weather
        </button>
      </form>

      <!-- Muestra Loading... mientras recibe los datos, es decir que loading esta a true hasta que pasa a false-->
      <div v-if="loading" class="mt-6 text-lg font-semibold animate-pulse">
        Loading...
      </div>

      <!-- Si todo va bien, muestro los datos que vienen en el objeto (promesa) -->
      <div v-else-if="weather"
        class="mt-6 p-6 bg-white text-gray-800 rounded-lg shadow-lg w-full max-w-md flex flex-col items-center">
        <h2 class="text-2xl font-semibold mb-4">
          Weather in {{ weather.name }}
        </h2>
        <img :src="getWeatherImage(weather.weather[0].description)" :alt="weather.weather[0].description"
          class="w-full h-48 object-cover rounded-lg mb-4" />
        <ul class="space-y-2 text-center">
          <li>
            <span class="font-semibold">Temperature:</span>
            {{ weather.main.temp }} °C
          </li>
          <li>
            <span class="font-semibold">Weather:</span>
            {{ weather.weather[0].description }}
          </li>
          <li>
            <span class="font-semibold">Humidity:</span>
            {{ weather.main.humidity }}%
          </li>
          <li>
            <span class="font-semibold">Wind Speed:</span>
            {{ weather.wind.speed }} m/s
          </li>
        </ul>
      </div>

      <!-- Aqui muestro el error (si lo hay) que recojo previamente en catch (err) -->
      <div v-if="error" class="mt-6 text-red-500 text-lg font-semibold">
        {{ error }}
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      loading: false,
      location: "",
      weather: null,
      error: "",
    };
  },
  methods: {
    async getWeather() {
      // Seteando valores por defecto
      this.loading = false;
      this.error = "";
      this.weather = null;
      const apiKey = import.meta.env.VITE_API_KEY;
      const apiUrl = import.meta.env.VITE_API_URL;
      const requestUrl = `${apiUrl}?q=${this.location}&units=metric&appid=${apiKey}`;
      // Hacemos try / catch / finally para gestionar respuesta
      try {
        const response = await fetch(requestUrl);
        if (!response.ok) {
          throw new Error("Failed to fetch weather data. Please try again.");
        }
        const data = await response.json();
        this.weather = data;
      } catch (err) {
        this.error = err.message;
      } finally {
        this.loading = false;
      }
    },
    getWeatherImage(description) {
      const weatherImages = {
        "clear sky": "https://media.istockphoto.com/id/1433196601/photo/photograph-of-pure-blue-summer-sky.webp?b=1&s=612x612&w=0&k=20&c=492P1TfC5uiCCBnI41-NqdR9B0AKdHZ0V5OgHPzNONo=",
        "few clouds": "https://media.istockphoto.com/id/1497379890/photo/blue-sky-in-summer-sunlight-sunny-day.jpg?b=1&s=612x612&w=0&k=20&c=at3rQOcOzalcPnfi88NYeDEgdx4veqY66vaP0NDzN7c=&quot",
        "scattered clouds": "https://media.istockphoto.com/id/645173476/photo/cirrocumulus-clouds-cloudscape.jpg?b=1&s=612x612&w=0&k=20&c=-W26OegFB5dyClvddIleYBn-mnESy75oC2Kd7KA4p84=",
        "broken clouds": "https://media.istockphoto.com/id/889072364/photo/cloudscape.jpg?b=1&s=612x612&w=0&k=20&c=O2OSciz-6CiansW5bZkIuCkAQOtaRgcSyfF87fG4oGU=",
        "shower rain": "https://media.istockphoto.com/id/1476189983/photo/summer-rain-raindrops-bad-weather-depression.webp?b=1&s=612x612&w=0&k=20&c=ZvyERvWpyuJsHEUlxWmMyarHV_waxNjhqGV0ZSBppQQ=",
        "moderate rain": "https://media.istockphoto.com/id/1476189983/photo/summer-rain-raindrops-bad-weather-depression.webp?b=1&s=612x612&w=0&k=20&c=ZvyERvWpyuJsHEUlxWmMyarHV_waxNjhqGV0ZSBppQQ=",
        "rain": "https://media.istockphoto.com/id/1476189983/photo/summer-rain-raindrops-bad-weather-depression.jpg?b=1&s=612x612&w=0&k=20&c=ZvyERvWpyuJsHEUlxWmMyarHV_waxNjhqGV0ZSBppQQ=&quot",
        "thunderstorm": "https://media.istockphoto.com/id/517643357/photo/thunderstorm-lightning-with-dark-cloudy-sky.webp?b=1&s=612x612&w=0&k=20&c=5iNU1jL5V6Jsz8oHL_CnRYKnmvR0lm67yTNmTYEHKTA=",
        "snow": "https://media.istockphoto.com/id/535513443/photo/pedestrians-crossing-the-street-on-a-snowy-day.jpg?b=1&s=612x612&w=0&k=20&c=7LEDQE1PCvkL03znC6XKW7KXHhT0MN6OH6kFcboONuk=&quot",
        "mist": "https://media.istockphoto.com/id/1047226654/photo/fog-over-beech-forest-in-aramaio-alava.webp?b=1&s=612x612&w=0&k=20&c=dCK3TzRTuAP3u02Xv4f_3Uzds_s8Zs3eKm70imRNWu8=",
      };
      return weatherImages[description] || "https://cdn.pixabay.com/photo/2023/12/12/19/08/moutains-8445767_640.jpg";
    },
  },
};
</script>