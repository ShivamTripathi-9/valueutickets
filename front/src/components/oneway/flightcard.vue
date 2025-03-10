<script setup>
import { computed, watch, ref, onMounted } from "vue";
import { useFlightStore } from "@/stores/flightStore";
import { usePostDataStore } from "@/stores/postDataStore";
import { useRouter } from "vue-router";
import axios from "axios"; // Axios for API calls

const flightStore = useFlightStore();
const postDataStore = usePostDataStore();
const router = useRouter();
const loading = ref(true); // Loading state

// Reactive object to store IATA-to-City mapping
const airportData = ref({});

// Function to fetch and map airport data
const fetchAirportData = async () => {
  try {
    const response = await axios.get("https://crm.valueutickets.com/api/v1/airports/");
    response.data.forEach((airport) => {
      if (airport.iata && airport.city) {
        airportData.value[airport.iata] = airport.city; // Map IATA to City
      }
    });
    console.log("Mapped Airport Data:", airportData.value); // Debugging log
  } catch (error) {
    console.error("Error fetching airport data:", error);
  }
};

// Function to format City Name with IATA
const formatCityWithIATA = (iata) => {
  if (!iata) return "Unknown (N/A)"; // Handle missing IATA codes
  const city = airportData.value[iata];
  return city ? `${city} (${iata})` : `Unknown (${iata})`;
};

// Fetch airport data on component mount
onMounted(async () => {
  await fetchAirportData(); // Ensure airport data is fetched before rendering
  console.log("Flight data:", flights.value); // Log flights data for debugging
  if (!flights.value.length) {
    console.warn("No flight data found! Make sure it's stored before navigating.");
  }
});

// Function to format duration (minutes to hours & minutes)
const formatDuration = (minutes) => {
  const hours = Math.floor(minutes / 60);
  const mins = minutes % 60;
  return `${hours}h ${mins}m`;
};

// Function to format date-time
const formatDateTime = (dateTimeString) => {
  const date = new Date(dateTimeString);
  return new Intl.DateTimeFormat("en-US", {
    month: "short",
    day: "2-digit",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    hour12: true,
  }).format(date);
};

// Retrieve flight data from store
const flights = computed(() => {
  if (!flightStore.flightData) return [];
  // Sort flights by stops and then by price
  return [...flightStore.flightData].sort((a, b) => {
    if (a.stop_count === b.stop_count) {
      return a.price - b.price
    }
    return a.stop_count - b.stop_count;
  });
});

// Placeholder for airline logos
const airlineLogos = {
  jetBlue:
    "https://upload.wikimedia.org/wikipedia/commons/thumb/3/3c/JetBlue_Airways_Logo.svg/220px-JetBlue_Airways_Logo.svg.png",
  "American Airlines":
    "https://upload.wikimedia.org/wikipedia/en/thumb/2/23/American_Airlines_logo_2013.svg/1920px-American_Airlines_logo_2013.svg.png",
  Delta:
    "https://upload.wikimedia.org/wikipedia/commons/thumb/d/d1/Delta_logo.svg/220px-Delta_logo.svg.png",
  "United Airlines":
    "https://upload.wikimedia.org/wikipedia/en/thumb/e/e0/United_Airlines_Logo.svg/464px-United_Airlines_Logo.svg.png",
  "Frontier Airlines":
    "https://upload.wikimedia.org/wikipedia/commons/thumb/c/cf/Frontier_Airlines_logo.svg/220px-Frontier_Airlines_logo.svg.png",
  "Hawaiian Airlines":
    "https://upload.wikimedia.org/wikipedia/en/thumb/3/3b/Hawaiian_Airlines_logo_2017.svg/137px-Hawaiian_Airlines_logo_2017.svg.png",
  "Southwest Airlines":
    "https://upload.wikimedia.org/wikipedia/commons/thumb/6/6f/Southwest_Airlines_logo_2014.svg/220px-Southwest_Airlines_logo_2014.svg.png",
  "Alaska Airlines":
    "https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/Alaska_Airlines_logo.svg/220px-Alaska_Airlines_logo.svg.png",
};

// Default logo for undefined airlines
const defaultLogo = "https://via.placeholder.com/150?text=No+Logo";

const officeNumber = ["#09c398", "#F40000B5", "#BAA035"];

// Function to determine the color based on index
const selectColor = (index) => {
  return officeNumber[index % officeNumber.length];
};


// Method to handle flight booking
const bookFlight = (flight) => {
  router.push({
    path: "/pay",
    query: {
      flight_name: flight.flight_name,
      src: postDataStore.postdata?.source_iata,
      departure: flight.departure,
      dest: postDataStore.postdata?.destination_iata,
      arrival: flight.arrival,
      duration: flight.duration,
      stop_count: flight.stop_count,
      price: flight.price,
    },
  });
};

// Watch flights data to update loading state
watch(flights, (newFlights) => {
  if (newFlights.length > 0) {
    loading.value = false;
  }
});

// Color settings for styling
const flightColors = ["#09c398", "#F40000B5", "#BAA035"];
const getColor = (index) => flightColors[index % flightColors.length];

const buttonGradients = [
  "linear-gradient(to right, #3E41EC, #09C398)",
  "linear-gradient(to right, #3E41EC, #F4000061)",
];
const getGradient = (index) => buttonGradients[index % buttonGradients.length];
</script>

<template>
  <!-- Loading Skeleton -->
  <div v-if="loading" class="loading-container">
    <div class="loading-card" v-for="n in 3" :key="n">
      <div class="loading-bar"></div>
      <div class="loading-content"></div>
    </div>
  </div>
  <div v-else style="width: 100%">
    <div class="results-container">
      <!-- Flight Data -->
      <div
        v-for="(flight, index) in flights"
        :key="index"
        class="flight-card"
        :id="index + 1"
      >
        <div
          class="color-bar"
          :style="{ backgroundColor: getColor(index) }"
        ></div>

        <!-- Flight Details -->
        <div class="flight-details">
          <img
            :src="airlineLogos[flight.flight_name] || defaultLogo"
            alt="Airline Logo"
            class="airline-logo"
          />
                    <div class="departure">
            <i class="fas fa-plane-departure"></i>
            <p>
              <!-- <strong>Departure:</strong>&nbsp;From: -->
              <span style="font-size: 17px; font-weight: 700">
                {{ formatCityWithIATA(postDataStore.postdata?.source_iata) || "N/A" }}
              </span>
            </p>
            <p style="color: #969696; font-size: 15px; font-weight: 500">
              {{ formatDateTime(flight.departure) || "N/A" }}
            </p>
          </div>

          <div class="duration">
            <img src="/icons8-airplane-24 (1).png" />
            <span class="dashed"></span>
            <img src="/icons8-dot-24.png" />
          </div>

          <div class="arrival">
            <i class="fas fa-plane-arrival"></i>&nbsp;
            <p>
              <!-- <strong>Arrival:</strong>&nbsp;To: -->
              <span style="font-size: 17px; font-weight: 700">
                {{ formatCityWithIATA(postDataStore.postdata?.destination_iata) || "N/A" }}
              </span>
            </p>
            <p style="color: #969696; font-size: 15px; font-weight: 500">
              {{ formatDateTime(flight.arrival) || "N/A" }}
            </p>
          </div>
        </div>
        <hr style="width: auto" />
        <!-- Pricing & Stops -->
        <div class="pricing">
          <span
            class="office-number"
            :style="{ backgroundColor: selectColor(index) }"
          >
            <i class="fas fa-stopwatch"></i>&nbsp;
            <strong>Duration:</strong>
            &nbsp;{{ formatDuration(flight.duration) || "N/A" }}
          </span>
          <div>
            <p>
              &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<strong>Stops:</strong>
              {{ flight.stop_count || 0 }}
            </p>
          </div>
          <p :style="{ color: getColor(index) }" style="font-size: 22px">
            <strong>${{ flight.price || "N/A" }}</strong>
          </p>
          <button
            @click="bookFlight(flight)"
            class="book-now"
            :style="{ background: getGradient(index) }"
          >
            Book Now
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.duration {
  display: flex;
  align-items: center;
  justify-self: center;
  gap: 10px;
  width: 60%;
}
.dashed {
  display: flex;
  align-items: center;
  justify-self: center;
  width: 100%;
  border: 2px dashed black;
}
.results-container {
  width: 100%;
  margin: auto;
}

.flight-card {
  background-color: #fff;
  border: 1px solid #ddd;
  padding: 0%;
  margin: 10px 0;
  border-radius: 21px;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.1);
}

.color-bar {
  height: 50px;
  width: 100%;
  border-radius: 21px 21px 0 0;
}

.airline-logo {
  max-width: 100px;
  max-height: 50px;
}

.flight-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
}

.pricing {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-around;
  padding: 15px;
}

.office-number {
  background-color: #09c398;
  color: white;
  font-weight: 700;
  padding: 7px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  width: auto;
  margin-top: 10px;
}

.book-now {
  background-image: linear-gradient(to right, #3e41ec, #09c398);
  color: white;
  padding: 7px;
  font-weight: 700;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  width: 100px;
  margin-top: 10px;
  transform: scale(1.2);
  margin-right: 20px;
}

/* Responsive Styles */
@media (max-width: 620px) {
  .results-container {
    display: flex;
    flex-direction: column;
  }
  .duration {
    width: 100%;
  }
  .flight-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .flight-details {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .pricing {
    flex-direction: column;
    align-items: center;
  }

  .office-number,
  .book-now {
    width: 100%;
    max-width: 250px;
  }
}

/* Loading Skeleton */
.loading-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
  justify-content: center;
  text-align: center;
}

.loading-card {
  background: #f3f3f3;
  border-radius: 8px;
  padding: 20px;
  width: 100%;
  height: 150px;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.1);
  animation: pulse 1.5s infinite ease-in-out;
}

.loading-bar {
  height: 20px;
  width: 80%;
  background: #ddd;
  border-radius: 5px;
}

.loading-content {
  margin-top: 10px;
  width: 90%;
  height: 60px;
  background: #ddd;
  border-radius: 5px;
}

@keyframes pulse {
  0% {
    background-color: #f3f3f3;
  }
  50% {
    background-color: #e0e0e0;
  }
  100% {
    background-color: #f3f3f3;
  }
}
</style>

