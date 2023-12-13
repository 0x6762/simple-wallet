<template>
  <div class="wrap-wallet-address">
    <input
      type="text"
      placeholder="Input an address..."
      v-model="inputValue"
      @keyup.enter="updateWalletAddress"
    />
    <p class="wallet-address">{{ walletAddress }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from 'vue'

// Initialize walletAddress with value from local storage or an empty string
const walletAddress = ref(localStorage.getItem('walletAddress') || '')
const inputValue = ref('')

const updateWalletAddress = () => {
  walletAddress.value = inputValue.value
  localStorage.setItem('walletAddress', inputValue.value)
  inputValue.value = ''
}
</script>

<style lang="scss" scoped></style>
