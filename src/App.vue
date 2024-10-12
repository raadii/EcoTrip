<template>
  <div>
    <form @submit.prevent="getCarbonFootprint">
      <label for="distance">Distance (in km):</label>
      <input type="number" v-model="distance" id="distance" required />

      <label for="vehicle">Vehicle Type:</label>
      <select v-model="vehicle" id="vehicle" required>
        <option value="SmallDieselCar">Small Diesel Car</option>
        <option value="MediumPetrolCar">Medium Petrol Car</option>
        <option value="LargeHybridCar">Large Hybrid Car</option>
        <!-- Add more options as needed -->
      </select>

      <button type="submit">Calculate Carbon Footprint</button>
    </form>

    <div v-if="carbonFootprint">
      <h3>Carbon Footprint:</h3>
      <p>{{ carbonFootprint }} kg CO₂</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from "axios";

const distance = ref("");
const vehicle = ref("SmallDieselCar");
const carbonFootprint = ref(null);

async function getCarbonFootprint() {
  const options = {
    method: "GET",
    url: `https://carbonfootprint1.p.rapidapi.com/CarbonFootprintFromCarTravel`,
    params: {
      distance: distance.value,
      vehicle: vehicle.value,
    },
    headers: {
      "x-rapidapi-host": "carbonfootprint1.p.rapidapi.com",
      "x-rapidapi-key": "d19d1bbdd1mshad2faac6f37714ep1450c1jsn794651fe08c2", // Replace with your actual API key
    },
  };

  try {
    const response = await axios.request(options);
    carbonFootprint.value = response.data.carbonEquivalent;
  } catch (error) {
    console.error("Error fetching carbon footprint:", error);
  }
}
</script>


<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
