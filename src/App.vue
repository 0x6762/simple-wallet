<template>
  <div>
    MATIC Balance: {{ maticBalance }}
    <br />
    <div v-for="(balance, token) in tokenBalances" :key="token">
      {{ token }} Balance: {{ balance.amount }} ({{ balance.usdValue }} USD)
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import Web3 from 'web3'
import axios from 'axios'
import { erc20Abi } from './components/erc20abi.js'

export default defineComponent({
  setup() {
    const maticBalance = ref('Loading...')
    const tokenBalances = ref({})

    onMounted(async () => {
      const web3 = new Web3('https://rpc-mainnet.maticvigil.com')
      const address = '0xc834bD2C217835E770b3Ba3d6c1D38eD45d5c291'
      const contractAddresses = [
        '0x6d80113e533a2C0fe82EaBD35f1875DcEA89Ea97'
        // Add other contract addresses here
      ]

      const balanceWei = await web3.eth.getBalance(address)
      maticBalance.value = web3.utils.fromWei(balanceWei, 'ether')

      for (const contractAddress of contractAddresses) {
        const contract = new web3.eth.Contract(erc20Abi, contractAddress)
        const tokenName = await contract.methods.name().call()
        const balanceWei = await contract.methods.balanceOf(address).call()
        let balance = web3.utils.fromWei(balanceWei, 'ether')

        // Format the balance to display only three decimal places
        balance = parseFloat(balance).toFixed(3)

        // Convert the token name to a format that matches the CoinGecko API coin IDs
        const coinId = tokenName.replace(/\s+/g, '-').toLowerCase()

        // Fetch the current price of the token in USD
        let priceInUsd = 0
        try {
          const response = await axios.get(
            `https://api.coingecko.com/api/v3/simple/price?ids=${coinId}&vs_currencies=usd`
          )
          console.log(`Coin ID: ${coinId}`, response.data) // Update this line
          priceInUsd = response.data[coinId]?.usd
        } catch (error) {
          console.error(`Error fetching price for coin ${coinId}:`, error)
        }

        if (!priceInUsd) {
          console.error(`Unable to fetch price for coin ${coinId}`)
          continue
        }

        // Calculate the balance in USD
        const usdValue = parseFloat(balance) * priceInUsd

        if (parseFloat(balance) > 0) {
          tokenBalances.value[tokenName] = { amount: balance, usdValue: usdValue.toFixed(2) }
        }
      }
    })

    return {
      maticBalance,
      tokenBalances
    }
  }
})
</script>

<style scoped></style>
