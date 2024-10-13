<template>
  <!-- Hero Section -->
  <div class="hero-section">
    <h1 class="main-title">
      <span class="eco">ECO</span><span class="trip">TRIP</span>
    </h1>
    <h2 class="subtitle">Travel more green</h2>
    <p class="subtitle2">
      Enter origin, destination, and vehicle information <br />
      to calculate <span class="carbon_footprint_subtitle2">carbon footprint</span> of trip
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
          <h1 class="calc-title">Calculate Your Travel Impact</h1>
          <h3 class="calc-subtitle">
            Track your carbon footprint and make greener travel choices
          </h3>

          <form @submit.prevent="getCarbonFootprint">
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
            <button type="submit" class="calc-button">CALCULATE</button>
          </form>
        </div>

        <!-- Map Section -->
        <div id="map" class="map-container"></div>
      </div>

      <!-- Div2: Output Section -->
      <div id="div2" class="output-container">
        <!-- Carbon Footprint Section -->
        <h3>
          Your Carbon Footprint is
          <span v-if="carbonFootprint">{{ carbonFootprint.toFixed(2) }}</span>
          <span v-else>___</span>
          <!-- Placeholder when the value is not calculated yet -->
          kg CO₂
        </h3>

        <!-- Trees Equivalent Section -->
        <h1>
          That is equivalent to
          <span v-if="trees">{{ trees.toFixed(2) }}</span>
          <span v-else>___</span>
          <!-- Placeholder for trees -->
          trees!
        </h1>
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
html {
  padding: 50px 0;
}
.hero-section {
  background: linear-gradient(#EBEED6, #CCE6BB);
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
  font-size: 100px;
  font-weight: bold;
  margin-bottom: -35px;
}

.subtitle {
  color: #5d9835;
  font-size: 30px;
  margin-bottom: 30px;
}

.subtitle2 {
  font-family: "Ropa Sans", sans-serif;
  font-size: 30px;
  color: #c5803d;
  line-height: 1.5;
}

.carbon_footprint_subtitle2 {
  color: #5d9835;
}

.scroll-button {
  background: linear-gradient(to right,#BE7F41, #583B1E);
  color: white;
  border: none;
  padding: 10px 80px;
  font-size: 20px;
  margin-top: 40px;
  cursor: pointer;
  border-radius: 10px;
}

.scroll-button:hover {
  background: linear-gradient(to right,#754f29, #432d18);
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  margin-top: 20px;
}

.grid-item {
  display: flex;
  flex-direction: column;
}
.calc-section {
  background: linear-gradient(#CCE6BB, #BBDFA3);
  flex-direction: column;
  align-items: center;
  padding: 80px;
} 

.form-map-container {
  padding-top: 30px;
  display: flex;
  justify-content: space-between;
  background-color: #d5a77b;
  padding: 20px;
  border-radius: 15px;
}
#map {
  max-width: 700px;
  max-height: 300px;
  margin-top: 20px;
  border-radius: 30px;
  flex: 1;
  background-color: #f0f0f0;
  margin-left: 20px;
  border-radius: 50px;
  justify-content: center;
  align-items: center;
}

.form-container {
  flex: 1;
  max-width: 50%;
  justify-content: center;
  align-items: center;
}

.calc-title {
  color: white;
  text-align: left;
}
.calc-subtitle {
  color: white;
  text-align: left;
  font-size: 1.5rem;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* 2 columns */
  grid-gap: 10px;
}

.grid-item input,
select {
  width: 100%;
  padding: 20px;
  background-color: #bbdfa3;
  border: none;
  border-radius: 10px;
  font-size: 1.2rem;
}

.calc-button {
  margin-top: 30px;
  padding: 24px 60px;
  background-color: #46392d;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 2rem;
  background: linear-gradient(to right, #a5e27c, #ffaa57 100%);
}

.calc-button:hover {
  background: linear-gradient(to right, #89b967, #ae7237);
  color: white;
}

.output-container {
  margin-top: 10px;
  background-color: #e8f4e1;
  padding: 20px;
  border-radius: 10px;
  font-size: 2rem;
  align-items: center;
  justify-content: center;
}

h1 {
  text-align: center;
  font-size: 3rem;
  margin-bottom: 0.5rem;
  color: black;
  font-family: "Ropa Sans", sans-serif;
  font-weight: 400;
  font-style: normal;
}

h3 {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 1rem;
  color: black;
  font-family: "Ropa Sans", sans-serif;
  font-weight: 400;
  font-style: normal;
}
</style>
