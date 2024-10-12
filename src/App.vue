<template>
  <div>
    <form @submit.prevent="getCarbonFootprint">
      <label for="vehicle">Vehicle Type:</label>
      <select v-model="vehicle" id="vehicle" required>
        <option value="SmallDieselCar">Small Diesel Car</option>
        <option value="MediumPetrolCar">Medium Petrol Car</option>
        <option value="LargeHybridCar">Large Hybrid Car</option>
      </select>

      <!-- Input fields for origin and destination -->
      <div>
        <label for="origin">Origin:</label>
        <input type="text" id="origin" placeholder="Enter origin" required />
      </div>
      <div>
        <label for="destination">Destination:</label>
        <input
          type="text"
          id="destination"
          placeholder="Enter destination"
          required
        />
      </div>

      <div id="map"></div>
      <!-- Google Map will be rendered here -->

      <button type="submit">Calculate Carbon Footprint</button>
    </form>

    <div v-if="carbonFootprint">
      <h3>Carbon Footprint:</h3>
      <p>{{ carbonFootprint }} kg CO₂</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { onMounted } from "vue";

const vehicle = ref("SmallDieselCar");
const carbonFootprint = ref(null);
const distance = ref(null); // Store the distance from the map here

// Function to initialize the map once Google Maps is loaded
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
        travelMode: "DRIVING",
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
  console.log("Vehicle Type:", vehicle.value);

  // Make API request to get carbon footprint
  try {
    const response = await axios.get(
      `https://carbonfootprint1.p.rapidapi.com/CarbonFootprintFromCarTravel`,
      {
        params: {
          distance: distance.value, // Distance in km
          vehicle: vehicle.value, // Vehicle type
        },
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
  width: 100vh;
  height: 600px;
  margin-top: 20px;
}
</style>
