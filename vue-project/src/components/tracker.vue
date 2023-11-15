<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'

const cryptoList = ref([])
const error = ref('')
const isLoading = ref(true) // Add a loading state
const searchTerm = ref('')
const selectedCrypto = ref(null)

onMounted(async () => {
  await fetchCryptoList()
})

const fetchCryptoList = async () => {
  try {
    const response = await axios.get('https://api.coingecko.com/api/v3/coins/markets', {
      params: {
        vs_currency: 'usd',
        order: 'market_cap_desc',
        per_page: 10,
        page: 1,
        sparkline: false
      }
    })
    cryptoList.value = response.data
  } catch (error) {
    console.error('Error fetching cryptocurrency data:', error)
    setError('Failed to fetch cryptocurrency data. Please try again later.')
  } finally {
    // Set isLoading to false regardless of success or failure
    isLoading.value = false
  }
}

const setError = (errorMessage: string) => {
  error.value = errorMessage
}

const filteredCryptoList = ref([])

// Watch for changes in the searchTerm and update the filteredCryptoList accordingly
watch(
  () => {
    const normalizedSearchTerm = searchTerm.value.toLowerCase()
    filteredCryptoList.value = cryptoList.value.filter(
      (crypto) =>
        crypto.name.toLowerCase().includes(normalizedSearchTerm) ||
        crypto.symbol.toLowerCase().includes(normalizedSearchTerm)
    )
  },
  { immediate: true }
) // Add immediate: true to trigger the watch immediately

// Show detailed view for a selected cryptocurrency
const showDetails = (crypto) => {
  selectedCrypto.value = crypto
}

// Close the detailed view modal
const closeDetails = () => {
  selectedCrypto.value = null
}
</script>

<template>
  <div>
    <h1>Crypto Tracker</h1>

    <div>
      <input v-model="searchTerm" placeholder="Search cryptocurrencies" />
    </div>

    <div v-if="isLoading" class="loading">
      <p>Loading...</p>
    </div>

    <div v-else>
      <div v-if="error">
        <p class="error">{{ error }}</p>
      </div>

      <div v-else>
        <ul>
          <li v-for="crypto in filteredCryptoList" :key="crypto.id">
            <div @click="showDetails(crypto)">
              <p>{{ crypto.name }} ({{ crypto.symbol }})</p>
              <p>Price: ${{ crypto.current_price }}</p>
              <p>Market Cap: ${{ crypto.market_cap }}</p>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- Detailed view modal -->
    <div v-if="selectedCrypto" class="modal" @click="closeDetails">
      <div class="modal-content">
        <span class="close" @click="closeDetails">&times;</span>
        <h2>{{ selectedCrypto.name }} ({{ selectedCrypto.symbol }})</h2>
        <p>Price: ${{ selectedCrypto.current_price }}</p>
        <p>Market Cap: ${{ selectedCrypto.market_cap }}</p>
      </div>
    </div>
  </div>
</template>

<style>
div {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  background-color: #f5f5f5;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

p {
  margin: 0;
  color: #555;
}

.error {
  color: red;
}

.loading {
  text-align: center;
  font-size: 18px;
  color: #777;
}

input {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

/* Modal styles */
.modal {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgb(0, 0, 0);
  background-color: rgba(0, 0, 0, 0.4);
}

.modal-content {
  background-color: #fefefe;
  margin: 15% auto;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
}
</style>
