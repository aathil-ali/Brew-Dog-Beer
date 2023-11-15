<template>
  <div
    class="relative flex items-top justify-center min-h-screen bg-gray-100 sm:items-center sm:pt-0"
  >
    <div class="max-w-4xl mx-auto sm:px-6 lg:px-8">
      <a
        class="flex justify-center pt-8 sm:pt-0"
        href="https://nuxtjs.org"
        target="_blank"
      >
      </a>
      <h1 class="mb-4 d-flex justify-content-center title-head">
        Brewdog Beers
      </h1>
    </div>

    <!-- Dropdown for sorting -->
    <b-row class="mb-3 sort">
      <!-- Sorting dropdown for ABV -->
      <b-col lg="6">
        <label for="sortAbv" class="form-label">Sort by ABV:</label>
        <b-form-select v-model="sortOptionByAbv" id="sortAbv">
          <option value="abv">Ascending</option>
          <option value="abvDesc">Descending</option>
        </b-form-select>
      </b-col>

      <!-- Sorting dropdown for content -->
      <b-col lg="6" class="text-lg-end">
        <label for="sortContent" class="form-label">Sort by Content: Dry Hopped / Lactose </label>
        <b-form-select v-model="sortOptionByContent" id="sortContent">
          <option value="all">All</option>
          <option value="dryHop">Contains Dry Hopped</option>
          <option value="lactose">Contains Lactose</option>
        </b-form-select>
      </b-col>
    </b-row>

    <b-row class="main">
      <b-col
        v-for="beer in paginatedBeers"
        :key="beer.id"
        :lg="4"
        :md="6"
        :sm="12"
        class="mb-4"
      >
        <b-card class="bg-white overflow-hidden shadow sm:rounded-lg p-6">
          <img
            :src="beer.image_url || defaultImageUrl"
            alt="Beer Image"
            class="card-img-top mx-auto mb-2"
            style="max-height: 350px; object-fit: contain"
          />
          <b-card-body>
            <h5 class="card-title">{{ beer.name }}</h5>
            <p class="card-text">
              <strong>Tagline:</strong> {{ beer.tagline }}
            </p>
            <p class="card-text description">
              <strong>Description:</strong> {{ beer.description }}
            </p>
            <p class="card-text"><strong>ABV:</strong> {{ beer.abv }}%</p>
            <p class="card-text"><strong>IBU:</strong> {{ beer.ibu }}</p>
            <p
              v-if="
                beer?.ingredients?.hops?.some(
                  (hop) => hop.name.toLowerCase() === 'centennial'
                )
              "
            >
              <strong>Contains Centennial Hops</strong>
            </p>
            <p
              v-if="
                beer?.ingredients?.hops?.some(
                  (hop) => hop.name.toLowerCase() === 'lactose'
                )
              "
            >
              <b-badge variant="danger">
                <strong>Contains Lactose</strong></b-badge
              >
            </p>
            <p
              v-if="
                beer?.ingredients?.hops?.some(
                  (hop) => hop.add.toLowerCase() === 'dry hop'
                )
              "
            >
              <b-badge variant="primary">
                <strong>Dry Hopped </strong>
              </b-badge>
            </p>
          </b-card-body>
        </b-card>
      </b-col>
    </b-row>
    <div class="d-flex justify-content-center mt-4">
      <pagination
        :records="totalPages"
        v-model="currentPage"
        @change="changePage"
        :per-page="pageSize"
        class="mt-4"
      />
    </div>
  </div>
</template>

<script>
// Importing necessary components and libraries
import Pagination from "vue-pagination-2";
import axios from "axios";
import defaultImage from "~/assets/default.jpeg"; // Import your default image

export default {
  // Vue component options
  components: {
    Pagination,
  },
  data() {
    // Data properties
    return {
      beers: [], // Array to store beer data
      currentPage: 1, // Current page for pagination
      pageSize: 9, // Number of beers to display per page
      defaultImageUrl: defaultImage, // Default image for beers without an image
      sortOptionByAbv: "abv", // Default sorting option for ABV
      sortOptionByContent: "all", // Default sorting option for beer content
    };
  },
  computed: {
    // Computed properties
    paginatedBeers() {
      // Returns a subset of beers based on the current page and page size
      const startIndex = (this.currentPage - 1) * this.pageSize;
      const endIndex = startIndex + this.pageSize;
      return this.beers.slice(startIndex, endIndex);
    },
    totalPages() {
      // Returns the total number of beers
      return this.beers.length;
    },
  },
  methods: {
    // Methods
    async fetchBeers() {
      try {
        let data;
        this.beers = [];
        // Fetch beer data from Punk API and filter out beers with Centennial hops
        for (let index = 1; index < 6; index++) {
          const response = await axios.get(
            `${this.$config.baseUrl}?page=${index}&per_page=80`
          );
          data = response.data.filter(
            (beer) =>
              !beer.ingredients.hops.some(
                (hop) => hop.name.toLowerCase() === "centennial"
              )
          );
          this.beers = [...this.beers, ...data];
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    async changePage(newPage) {
      // Handles page change events (currently empty)
      try {
        // Additional logic can be added if needed
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
  },
  watch: {
    // Watchers
    currentPage(newPage) {
      // Triggers the fetchBeers method when the current page changes
      // this.fetchBeers(newPage);
    },
    sortOptionByAbv(newOption) {
      // Sorts the beer list based on the selected ABV sorting option
      const sorted = this.beers.sort((a, b) => {
        const abvA = a.abv || 0;
        const abvB = b.abv || 0;

        if (this.sortOptionByAbv === "abvDesc") {
          // Sort by ABV in descending order
          return abvB - abvA;
        } else {
          // Sort by ABV in ascending order (default)
          return abvA - abvB;
        }
      });
      this.beers = sorted;
    },
    async sortOptionByContent(newOption) {
      // Filters the beer list based on the selected content sorting option
      try {
        alert(this.currentPage );
        this.currentPage = 1 ;
        await this.fetchBeers();

        let data;

        if (this.sortOptionByContent == "lactose") {
          data = this.beers.filter((beer) =>
            beer.ingredients.hops.some(
              (hop) => hop.name.toLowerCase() === "lactose"
            )
          );
        } else if (this.sortOptionByContent == "dryHop") {
          data = this.beers.filter((beer) =>
            beer.ingredients.hops.some(
              (hop) => hop.add.toLowerCase() === "dry hop"
            )
          );
        } else {
          data = this.beers;
        }

        this.beers = data;
        this.currentPage = 1 ;
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
  },
  // Lifecycle hook
  async mounted() {
    // Calls the fetchBeers method on component mount to initialize the beer data
    this.fetchBeers(this.currentPage);
  },
};
</script>
<style scoped>
.beer-card {
  border: 1px solid #ccc;
  padding: 15px;
  margin: 15px 0;
}
.main {
  padding: 10%;
}
p.card-text.description {
  max-height: 100px;
  overflow: hidden;
}
h5.card-title {
  min-height: 50px;
}
.sort {
  padding: 3%;
  max-width: 70%;
  margin: 0 auto;
}
.title-head {
  padding: 5%;
}
</style>