<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        v-model="searchQuery"
        @input="getSearchResults"
        type="text"
        placeholder="Search for a city"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="(mapboxSearchResult && mapboxSearchResult.length) || searchError"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">{{ searchError }}</p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResult"
            :key="searchResult.id"
            class="py-2 cursor-pointer text-white"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";

// Use the environment variable for the Mapbox API key
const mapboxApiKey = import.meta.env.VITE_MAPBOX_API_KEY;

const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResult = ref(null);

// Error handling
const searchError = ref("");

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json`,
          {
            params: {
              access_token: mapboxApiKey,
              types: "place",
            },
          }
        );
        if (result.data.features.length === 0) {
          searchError.value = "No city found! Try a different term.";
          mapboxSearchResult.value = null;
        } else {
          mapboxSearchResult.value = result.data.features;
          searchError.value = "";
        }
        console.log(mapboxSearchResult.value);
      } catch (error) {
        console.error(error);
        searchError.value = "Error finding a city.";
        mapboxSearchResult.value = null;
      }
    } else {
      mapboxSearchResult.value = null;
      searchError.value = "";
    }
  }, 300);
};
</script>
