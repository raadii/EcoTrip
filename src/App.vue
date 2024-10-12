<template>
  <div>
    <form @submit.prevent="getCarbonFootprint">
      <label for="vehicle">Travel Type:</label>
      <select v-model="endpoint" id="vehicle" required>
        <option value="CarbonFootprintFromCarTravel">Car</option>
        <option value="CarbonFootprintFromFlight">Flight</option>
        <option value="CarbonFootprintFromPublicTransit">Public Transit</option>
      </select>

      <label v-if="endpoint === 'CarbonFootprintFromCarTravel'" for="carType">Car Type:</label>
      <select v-if="endpoint === 'CarbonFootprintFromCarTravel'" v-model="carType" id="carType" required>
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

      <label v-if="endpoint === 'CarbonFootprintFromFlight'" for="flightType">Flight Type:</label>
      <select v-if="endpoint === 'CarbonFootprintFromFlight'" v-model="type" id="flightType" required>
        <option value="DomesticFlight">Domestic Flight</option>
        <option value="ShortEconomyClassFlight">Short Economy Class Flight</option>
        <option value="ShortBusinessClassFlight">Short Business Class Flight</option>
        <option value="LongEconomyClassFlight">Long Economy Class Flight</option>
        <option value="LongPremiumClassFlight">Long Premium Class Flight</option>
        <option value="LongBusinessClassFlight">Long Business Class Flight</option>
        <option value="LongFirstClassFlight">Long First Class Flight</option>
      </select>

      <label v-if="endpoint === 'CarbonFootprintFromPublicTransit'" for="transitType">Transit Type:</label>
      <select v-if="endpoint === 'CarbonFootprintFromPublicTransit'" v-model="type" id="transitType" required>
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

      <!-- Input fields for origin and destination -->
      <div>
        <label for="origin">Origin:</label>
        <input type="text" id="origin" placeholder="Enter origin" required />
      </div>
      <div>
        <label for="destination">Destination:</label>
        <input type="text" id="destination" placeholder="Enter destination" required />
      </div>

      <div id="map"></div>
      <!-- Google Map will be rendered here -->

      <button type="submit">Calculate Carbon Footprint</button>
    </form>

    <div v-if="carbonFootprint">
      <h3>Carbon Footprint:</h3>
      <p>{{ carbonFootprint }} kg CO₂</p>
    </div>
    <div v-if="trees">
      <h3>Trees Equivalent:</h3>
      <p>{{ trees }} trees</p>
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
  // Create Autocomplete instances for origin and destination
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
  const directionsRenderer = new google.maps.DirectionsRenderer({
    map: map,
  });

  // Function to calculate distance
  function calculateDistance() {
    const originPlace = originInput.getPlace();
    const destinationPlace = destinationInput.getPlace();

    if (originPlace && destinationPlace) {
      const request = {
        origin: originPlace.geometry.location,
        destination: destinationPlace.geometry.location,
        travelMode: endpoint.value === "CarbonFootprintFromPublicTransit" ? "TRANSIT" : "DRIVING",
      };

      directionsService.route(request, (result, status) => {
        if (status === "OK") {
          directionsRenderer.setDirections(result);
          const route = result.routes[0].legs[0];
          distance.value = route.distance.value / 1000; // distance in km
        } else {
          console.error("Directions request failed due to " + status);
        }
      });
    }
  }

  // Listen for changes in place selection
  originInput.addListener("place_changed", calculateDistance);
  destinationInput.addListener("place_changed", calculateDistance);
}

// Function to calculate carbon footprint and log the distance and vehicle type
async function getCarbonFootprint() {
  console.log("Distance:", distance.value ? distance.value : "Not available");
  console.log("Endpoint:", endpoint.value);
  
  // Determine the parameters based on the selected endpoint
  const params = {
    distance: distance.value, // Distance in km
  };

  // Set the vehicle or type based on the selected endpoint
  if (endpoint.value === "CarbonFootprintFromCarTravel") {
    params.vehicle = carType.value; // For cars, use vehicle type
  } else if (endpoint.value === "CarbonFootprintFromFlight" || endpoint.value === "CarbonFootprintFromPublicTransit") {
    params.type = type.value; // For flights and transit, use type
  }

  // Make API request to get carbon footprint
  try {
    const response = await axios.get(
      `https://carbonfootprint1.p.rapidapi.com/${endpoint.value}`,
      {
        params: params,
        headers: {
          "x-rapidapi-host": "carbonfootprint1.p.rapidapi.com",
          "x-rapidapi-key":
            "d19d1bbdd1mshad2faac6f37714ep1450c1jsn794651fe08c2",
        },
      }
    );

    // Update carbon footprint from the API response
    carbonFootprint.value = response.data.carbonEquivalent; // Adjust based on the actual API response structure
    console.log("Carbon Footprint:", carbonFootprint.value);
  } catch (error) {
    console.error("Error fetching carbon footprint:", error);
  }

  await getTrees();
}

// Function to calculate carbon footprint and log the distance and vehicle type
async function getTrees() {
  if (!carbonFootprint.value) return; // Do not proceed if carbon footprint is not available

  console.log("Calculating trees equivalent for:", carbonFootprint.value);
  
  // Prepare parameters for the API request
  const params = {
    weight: carbonFootprint.value, // Carbon footprint in kg
    unit: "kg"
  };

  // Make API request to get the tree equivalent
  try {
    const response = await axios.get(
      'https://carbonfootprint1.p.rapidapi.com/TreeEquivalent',
      {
        params: params,
        headers: {
          "x-rapidapi-host": "carbonfootprint1.p.rapidapi.com",
          "x-rapidapi-key":
            "d19d1bbdd1mshad2faac6f37714ep1450c1jsn794651fe08c2",
        },
      }
    );

    // Update the trees variable from the API response
    trees.value = response.data.numberOfTrees; // Adjust based on the actual API response structure
    console.log("Trees Equivalent:", trees.value);
  } catch (error) {
    console.error("Error fetching trees equivalent:", error);
  }
}

// Check if the Google Maps API is loaded, then initialize the map
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

onMounted(() => {
  loadGoogleMapsApi(); // Call the function to load the map
});
</script>

<style scoped>
#map {
  width: 100%;
  height: 600px;
  margin-top: 20px;
}
</style>