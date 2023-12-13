<template>
  <div class="wrap-wallet-address">
    <input
      type="text"
      placeholder="Input an address..."
      v-model="inputValue"
      @keyup.enter="updateWalletAddress"
    />
    <div class="wrap-content">
      <p class="wallet-name">My wallet</p>
      <p class="wallet-address">
        {{ walletAddress.slice(0, 5) + '...' + walletAddress.slice(-5) }}
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

// Initialize walletAddress with value from local storage or an empty string
const walletAddress = ref(localStorage.getItem('walletAddress') || '')
const inputValue = ref('')

const updateWalletAddress = () => {
  walletAddress.value = inputValue.value
  localStorage.setItem('walletAddress', inputValue.value)
  inputValue.value = ''
}
</script>

<style lang="scss" scoped>
.wrap-content {
  display: flex;
  align-items: center;
  margin-top: 24px;
  padding: 16px;
  gap: 16px;

  .wallet-name {
    font-size: 20px;
    font-weight: 600;
    color: var(--white);
  }
  .wallet-address {
    font-size: 14px;
    font-weight: 400;
    color: var(--gray600);
    border: 1px solid var(--color-surface);
    padding: 8px 12px;
    border-radius: 100px;
  }
}
</style>
