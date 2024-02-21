<template>
  <div class="wrap-wallet-address">
    <div class="wrap-input" v-if="!walletAddress">
      <input
        type="text"
        placeholder="Input an address..."
        v-model="inputValue"
        @keyup.enter="updateWalletAddress"
      />
    </div>

    <div class="wrap-content" v-else>
      <p class="wallet-name">My wallet</p>
      <p class="wallet-address">
        {{ walletAddress.slice(0, 5) + '...' + walletAddress.slice(-5) }}
      </p>
      <SettingsIcon class="btn-remove" @click="removeWalletAddress" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import SettingsIcon from '../assets/ui-icons/ic_settings.svg'

// Initialize walletAddress with value from local storage or an empty string
const walletAddress = ref(localStorage.getItem('walletAddress') || '')
const inputValue = ref('')

const updateWalletAddress = () => {
  walletAddress.value = inputValue.value
  localStorage.setItem('walletAddress', inputValue.value)
  inputValue.value = ''
  window.location.reload()
}

const removeWalletAddress = () => {
  localStorage.removeItem('walletAddress')
  walletAddress.value = ''
  window.location.reload()
}
</script>

<style lang="scss" scoped>
.wrap-input {
  input {
    width: 100%;
    border-radius: 16px;
    padding: 16px 24px;
    background-color: var(--color-surface);
    font-size: 14px;
    color: var(--white);
    &::placeholder {
      color: var(--gray600);
    }
    &:focus::placeholder {
      color: transparent;
    }
  }
}
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
  .btn-remove {
    width: 20px;
    height: 20px;
    stroke: var(--gray400);
    cursor: pointer;

    &:hover {
      stroke: var(--gray700);
    }
  }
}
</style>
