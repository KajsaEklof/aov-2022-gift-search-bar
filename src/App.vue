<script setup>
import { ref, watch, computed } from 'vue'
import { debounce } from 'lodash'
import LoadingSpinner from './LoadingSpinner.vue'

const searchTerm = ref('')
const isLoading = ref(false)
const products = ref([])
const hasProducts = computed(() => products.value.length > 0)

const findProducts = async term => {
  // Make sure the search field has a value before trying to display the search results
  if (searchTerm.value !== '') {
    try {
      const call = await fetch(`https://dummyjson.com/products/search?q=${term}`)
      const result = await call.json()

      products.value = result.products.slice(0, 10)
    } catch (err) {
      // use native `alert()` to show a generic error message
      alert(err)
    } finally {
      setTimeout(() => {
        isLoading.value = false
      }, 500)
    }
  }
}

const resetResults = () => {
  isLoading.value = false
  products.value = []
}

const debouncedFindProducts = debounce(findProducts, 300)

watch(searchTerm, newTerm => {
  if (newTerm === '') {
    resetResults()

    return
  }

  isLoading.value = true
  debouncedFindProducts(newTerm)
})
</script>

<template>
  <div class="w-full h-full flex flex-col gap-5 pt-8 items-center">
    <h1 class="text-4xl font-bold">Gift Search Bar</h1>
    <input type="text" class="p-2 border-2 border-gray-dark" v-model="searchTerm" placeholder="Start typing..." />
    <loading-spinner v-if="isLoading" />
    <template v-else>
      <ul v-if="hasProducts" class="list-disc">
        <li v-for="product in products" :key="product.id">{{ product.title }} - £{{ product.price }}</li>
      </ul>
      <span v-if="searchTerm !== '' && !hasProducts"> Sorry, no gifts found. Try searching for something else. </span>
    </template>
  </div>
</template>
