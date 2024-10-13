<template>
  <!-- Hero Section -->
  <div class="hero-section">
    <h1 class="main-title">
      <span class="eco">ECO</span><span class="trip">TRIP</span>
    </h1>
    <h2 class="subtitle">Travel more green</h2>
    <p class="subtitle2">
      Enter origin, destination, and vehicle information <br />
      to calculate carbon footprint of trip
    </p>
    <button @click="scrollToCalculation" class="scroll-button">
      Calculate
    </button>
  </div>

  <!-- Calculation Form -->
  <!-- Calculation Form -->
  <div class="calc-section">
    <div id="calculation-section" class="main-container2">
      <!-- Div1: Form and Map Section -->
      <div id="div1" class="form-map-container">
        <div class="form-container">
          <h1>Calculate Your Travel Impact</h1>
          <h3>Track your carbon footprint and make greener travel choices</h3>

          <form @submit.prevent="getCarbonFootprint">
            <label for="vehicle">Travel Type:</label>
            <div class="grid-container">
              <select v-model="endpoint" id="vehicle" required>
                <option value="CarbonFootprintFromCarTravel">Car</option>
                <option value="CarbonFootprintFromFlight">Flight</option>
                <option value="CarbonFootprintFromPublicTransit">
                  Public Transit
                </option>
              </select>
              <select
                v-if="endpoint === 'CarbonFootprintFromCarTravel'"
                v-model="carType"
                id="carType"
                required
              >
                <option value="SmallDieselCar">Small Diesel Car</option>
                <option value="MediumDieselCar">Medium Diesel Car</option>
                <option value="LargeDieselCar">Large Diesel Car</option>
                <option value="MediumHybridCar">Medium Hybrid Car</option>
                <option value="LargeHybridCar">Large Hybrid Car</option>
                <option value="MediumLPGCar">Medium LPG Car</option>
                <option value="LargeLPGCar">Large LPG Car</option>
                <option value="MediumCNGCar">Medium CNG Car</option>
                <option value="LargeCNGCar">Large CNG Car</option>
                <option value="SmallPetrolVan">Small Petrol Van</option>
                <option value="LargePetrolVan">Large Petrol Van</option>
                <option value="SmallDieselVan">Small Diesel Van</option>
                <option value="MediumDieselVan">Medium Diesel Van</option>
                <option value="LargeDieselVan">Large Diesel Van</option>
                <option value="LPGVan">LPG Van</option>
                <option value="CNGVan">CNG Van</option>
                <option value="SmallPetrolCar">Small Petrol Car</option>
                <option value="MediumPetrolCar">Medium Petrol Car</option>
                <option value="LargePetrolCar">Large Petrol Car</option>
                <option value="SmallMotorBike">Small Motor Bike</option>
                <option value="MediumMotorBike">Medium Motor Bike</option>
                <option value="LargeMotorBike">Large Motor Bike</option>
              </select>

              <select
                v-if="endpoint === 'CarbonFootprintFromFlight'"
                v-model="type"
                id="flightType"
                required
              >
                <option value="DomesticFlight">Domestic Flight</option>
                <option value="ShortEconomyClassFlight">
                  Short Economy Class Flight
                </option>
                <option value="ShortBusinessClassFlight">
                  Short Business Class Flight
                </option>
                <option value="LongEconomyClassFlight">
                  Long Economy Class Flight
                </option>
                <option value="LongPremiumClassFlight">
                  Long Premium Class Flight
                </option>
                <option value="LongBusinessClassFlight">
                  Long Business Class Flight
                </option>
                <option value="LongFirstClassFlight">
                  Long First Class Flight
                </option>
              </select>

              <select
                v-if="endpoint === 'CarbonFootprintFromPublicTransit'"
                v-model="type"
                id="transitType"
                required
              >
                <option value="Taxi">Taxi</option>
                <option value="ClassicBus">Classic Bus</option>
                <option value="EcoBus">Eco Bus</option>
                <option value="Coach">Coach</option>
                <option value="NationalTrain">National Train</option>
                <option value="LightRail">Light Rail</option>
                <option value="Subway">Subway</option>
                <option value="FerryOnFoot">Ferry On Foot</option>
                <option value="FerryInCar">Ferry In Car</option>
              </select>
              <div class="grid-item">
                <label for="origin">Origin:</label>
                <input
                  type="text"
                  id="origin"
                  placeholder="Enter origin"
                  required
                />
              </div>
              <div class="grid-item">
                <label for="destination">Destination:</label>
                <input
                  type="text"
                  id="destination"
                  placeholder="Enter destination"
                  required
                />
              </div>
            </div>
            <!-- <div id="map"></div> -->
            <button type="submit">Calculate Carbon Footprint</button>
          </form>
        </div>

        <!-- Map Section -->
        <div id="map" class="map-container"></div>
      </div>

      <!-- Div2: Output Section -->
      <div id="div2" class="output-container">
        <div v-if="carbonFootprint">
          <h3>Carbon Footprint:</h3>
          <p>{{ carbonFootprint }} kg CO₂</p>
        </div>
        <div v-if="trees">
          <h3>Trees Equivalent:</h3>
          <p>{{ trees }} trees</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const distance = ref(null);
const carType = ref("SmallDieselCar");
const type = ref("");
const endpoint = ref("CarbonFootprintFromCarTravel");
const carbonFootprint = ref(null);
const trees = ref(null);

function initializeMap() {
  const originInput = new google.maps.places.Autocomplete(
    document.getElementById("origin")
  );
  const destinationInput = new google.maps.places.Autocomplete(
    document.getElementById("destination")
  );
  const map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: 40.749933, lng: -73.98633 },
    zoom: 13,
  });
  const directionsService = new google.maps.DirectionsService();
  const directionsRenderer = new google.maps.DirectionsRenderer({ map: map });

  function calculateDistance() {
    const originPlace = originInput.getPlace();
    const destinationPlace = destinationInput.getPlace();
    if (originPlace && destinationPlace) {
      const request = {
        origin: originPlace.geometry.location,
        destination: destinationPlace.geometry.location,
        travelMode:
          endpoint.value === "CarbonFootprintFromPublicTransit"
            ? "TRANSIT"
            : "DRIVING",
      };
      directionsService.route(request, (result, status) => {
        if (status === "OK") {
          directionsRenderer.setDirections(result);
          const route = result.routes[0].legs[0];
          distance.value = route.distance.value / 1000;
        }
      });
    }
  }
  originInput.addListener("place_changed", calculateDistance);
  destinationInput.addListener("place_changed", calculateDistance);
}

async function getCarbonFootprint() {
  const params = { distance: distance.value };
  if (endpoint.value === "CarbonFootprintFromCarTravel") {
    params.vehicle = carType.value;
  } else if (
    endpoint.value === "CarbonFootprintFromFlight" ||
    endpoint.value === "CarbonFootprintFromPublicTransit"
  ) {
    params.type = type.value;
  }

  try {
    const response = await axios.get(
      `https://carbonfootprint1.p.rapidapi.com/${endpoint.value}`,
      {
        params: params,
        headers: {
          "x-rapidapi-host": "carbonfootprint1.p.rapidapi.com",
          "x-rapidapi-key":
            "3d075863femshdf64220b7079af1p1caf4fjsnade9f863b2c2",
        },
      }
    );
    carbonFootprint.value = response.data.carbonEquivalent;
  } catch (error) {
    console.error("Error fetching carbon footprint:", error);
  }
  await getTrees();
}

async function getTrees() {
  if (!carbonFootprint.value) return;
  const params = { weight: carbonFootprint.value, unit: "kg" };
  try {
    const response = await axios.get(
      "https://carbonfootprint1.p.rapidapi.com/TreeEquivalent",
      {
        params: params,
        headers: {
          "x-rapidapi-host": "carbonfootprint1.p.rapidapi.com",
          "x-rapidapi-key":
            "3d075863femshdf64220b7079af1p1caf4fjsnade9f863b2c2",
        },
      }
    );
    trees.value = response.data.numberOfTrees;
  } catch (error) {
    console.error("Error fetching trees equivalent:", error);
  }
}

function loadGoogleMapsApi() {
  if (typeof google !== "undefined") {
    initializeMap();
  } else {
    const checkGoogle = setInterval(() => {
      if (typeof google !== "undefined") {
        clearInterval(checkGoogle);
        initializeMap();
      }
    }, 100);
  }
}

function scrollToCalculation() {
  const element = document.getElementById("calculation-section");
  element.scrollIntoView({ behavior: "smooth" });
}

onMounted(() => {
  loadGoogleMapsApi();
});
</script>

<style scoped>
.hero-section {
  background-color: #e9edc9;
  width: 100vw;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.main-container {
  background-color: #e9edc9;
  width: 100vw;
  min-height: 100vh;
  padding: 20px;
}

.eco {
  color: #5d9835;
  font-weight: bold;
}

.trip {
  color: #c5803d;
  font-weight: bold;
}

.main-title {
  font-size: 90px;
  font-weight: bold;
  margin-bottom: 20px;
}

.subtitle {
  color: #5d9835;
  font-weight: bold;
}

.subtitle2 {
  font-family: "Ropa Sans", sans-serif;
  font-size: 45px;
  color: #c5803d;
}

.scroll-button {
  background-color: #c5803d;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 20px;
  margin-top: 20px;
  cursor: pointer;
  border-radius: 20px;
}

#map {
  max-width: 500px;
  height: 250px;
  margin-top: 20px;
  border-radius: 30px;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  margin-top: 20px;
}

.grid-item {
  display: flex;
  flex-direction: column;
}
.calc-section {
  background: linear-gradient(180deg, #cce6bb 0%, #bbdfa3 100%);
  width: 100vw;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}
.main-container2 {
  display: flex;
  flex-direction: column;
  width: 85vw;
  justify-content: space-between;
}

.form-map-container {
  display: flex;
  justify-content: space-between;
  background-color: #d5a77b;
  padding: 20px;
  border-radius: 15px;
}

.form-container {
  flex: 1;
  max-width: 50%;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* 2 columns */
  grid-gap: 10px;
}

.grid-item input,
select {
  width: 100%;
  padding: 10px;
  background-color: #bbdfa3;
  border: none;
  border-radius: 10px;
}

button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #ffffff;
  border: none;
  border-radius: 10px;
  cursor: pointer;
}

.map-container {
  flex: 1;
  background-color: #f0f0f0;
  margin-left: 20px;
  border-radius: 10px;
}

.output-container {
  margin-top: 20px;
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
}

h1 {
  font-size: 2rem;
  margin-bottom: 0.5rem;
  color: #fff;
}

h3 {
  font-size: 1.2rem;
  margin-bottom: 1rem;
  color: #fff;
}
</style>
