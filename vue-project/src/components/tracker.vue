<script setup lang="ts">
  import { ref, onMounted } from 'vue'
  import axios from 'axios'

  const cryptoList = ref([])
  const error = ref('')
  const isLoading = ref(true) // Add a loading state

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
  error.value = errorMessage;
}

</script>

<template>
  <div>
    <h1>Crypto Tracker</h1>

    <div v-if="isLoading">
      <p>Loading...</p>
    </div>

    <div v-else>
      <div v-if="error">
        <p class="error">{{ error }}</p>
      </div>

      <div v-else>
        <ul>
          <li v-for="crypto in cryptoList" :key="crypto.id">
            <p>{{ crypto.name }} ({{ crypto.symbol }})</p>
            <p>Price: ${{ crypto.current_price }}</p>
            <p>Market Cap: ${{ crypto.market_cap }}</p>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style>
  .error {
    color: red;
  }
</style> 