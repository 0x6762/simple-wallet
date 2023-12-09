<template>
  <div class="main">
    <div class="total-balance">
      <p class="label">Total balance</p>
      <p class="value">${{ totalBalance.toFixed(2) }}</p>
    </div>

    <ul class="token-list">
      <p>My tokens</p>
      <li v-for="(balance, token) in tokenBalances" :key="token">
        <div class="wrap-token">
          <img :src="`/icons/${balance.coinId}.svg`" alt="Token Icon" />
          <p class="token-name">{{ token }}</p>
        </div>
        <div class="wrap-token-value">
          <p class="token-value">{{ balance.amount }}</p>
          <p class="token-value-fiat">({{ balance.usdValue }} USD)</p>
        </div>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, computed } from 'vue'
import Web3 from 'web3'
import axios from 'axios'
import { erc20Abi } from './components/erc20abi.js'

export default defineComponent({
  setup() {
    const tokenBalances = ref({})

    onMounted(async () => {
      const web3 = new Web3('https://rpc-mainnet.maticvigil.com')
      // Wallet address to check the balance of
      const address = '0xc834bD2C217835E770b3Ba3d6c1D38eD45d5c291'
      //
      const contractAddresses = [
        '0x6d80113e533a2C0fe82EaBD35f1875DcEA89Ea97',
        '0xc2132D05D31c914a87C6611C10748AEb04B58e8F',
        '0x1d734A02eF1e1f5886e66b0673b71Af5B53ffA94'
        // Add other contract addresses here
      ]

      // Create a mapping between contract coinIds and CoinGecko coinIds
      const coinIdMapping = {
        '0x6d80113e533a2C0fe82EaBD35f1875DcEA89Ea97': 'aave-polygon-wmatic',
        '0xc2132D05D31c914a87C6611C10748AEb04B58e8F': 'tether',
        '0x1d734A02eF1e1f5886e66b0673b71Af5B53ffA94': 'stader'
        // Add more mappings as needed
      }

      // Create a mapping between contract addresses and token names
      const tokenNameMapping = {
        '0x6d80113e533a2C0fe82EaBD35f1875DcEA89Ea97': 'Aave WMATIC',
        '0xc2132D05D31c914a87C6611C10748AEb04B58e8F': 'USDT',
        '0x1d734A02eF1e1f5886e66b0673b71Af5B53ffA94': 'Stader'
        // Add more mappings as needed
      }

      // Get the MATIC balance of the address
      const balanceWei = await web3.eth.getBalance(address)
      let maticBalance = web3.utils.fromWei(balanceWei, 'ether')

      // Format the balance to display only three decimal places
      maticBalance = parseFloat(maticBalance).toFixed(3)

      // Fetch the current price of MATIC in USD
      let maticPriceInUsd = 0
      try {
        const response = await axios.get(
          `https://api.coingecko.com/api/v3/simple/price?ids=matic-network&vs_currencies=usd`
        )
        maticPriceInUsd = response.data['matic-network']?.usd
      } catch (error) {
        console.error(`Error fetching price for MATIC:`, error)
      }

      // Calculate the MATIC balance in USD
      const maticUsdValue = parseFloat(maticBalance) * maticPriceInUsd

      // Add MATIC balance to tokenBalances
      if (parseFloat(maticBalance) > 0) {
        tokenBalances.value['MATIC'] = {
          amount: maticBalance,
          usdValue: maticUsdValue.toFixed(2),
          coinId: 'matic-network'
        }
      }

      // Get the balance of each token
      for (const contractAddress of contractAddresses) {
        const contract = new web3.eth.Contract(erc20Abi, contractAddress)
        // Use the token name from the mapping
        const tokenName = tokenNameMapping[contractAddress]
        const balanceWei = await contract.methods.balanceOf(address).call()
        const decimals = await contract.methods.decimals().call()
        let balance
        if (contractAddress === '0xc2132D05D31c914a87C6611C10748AEb04B58e8F') {
          const ten = BigInt(10)
          const bigIntDecimals = BigInt(decimals)
          balance = Number(BigInt(balanceWei) / ten ** bigIntDecimals)
          balance = Number(balance.toFixed(10))
        } else {
          balance = web3.utils.fromWei(balanceWei, 'ether')
        }

        // Format the balance to display only three decimal places
        balance = (Math.floor(balance * 1e3) / 1e3).toFixed(3)

        // If the balance is zero, log a message and continue to the next token
        if (parseFloat(balance) === 0) {
          console.log(`No balance for token ${tokenName}`)
          continue
        }

        // Convert the contract address to a CoinGecko coinId using the mapping
        const coinId = coinIdMapping[contractAddress]

        // Fetch the current price of the token in USD
        let priceInUsd = 0
        try {
          const response = await axios.get(
            `https://api.coingecko.com/api/v3/simple/price?ids=${coinId}&vs_currencies=usd`
          )
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
          tokenBalances.value[tokenName] = {
            amount: balance,
            usdValue: usdValue.toFixed(2),
            coinId: coinId // Add this line
          }
        }
      }
    })

    const totalBalance = computed(() => {
      return Object.values(tokenBalances.value).reduce((total, balance) => {
        return total + parseFloat(balance.usdValue)
      }, 0)
    })

    return {
      tokenBalances,
      totalBalance
    }
  }
})
</script>

<style lang="scss" scoped>
.main {
  max-width: 600px;
  margin: 0 auto;
  padding: 8px;
}

.total-balance {
  background-color: var(--color-surface);
  margin-top: 16px;
  padding: 24px 24px;
  border-radius: 16px;
  .label {
    font-size: 14px;
    margin-bottom: 8px;
    color: var(--gray900);
  }
  .value {
    font-size: 32px;
    font-weight: 800;
  }
}

.token-list {
  list-style: none;
  margin-top: 24px;
  padding: 0 24px;

  li {
    align-items: center;
    display: flex;
    justify-content: space-between;
    margin-top: 16px;
    margin-bottom: 1rem;
    .wrap-token {
      display: flex;
      align-items: center;
      img {
        width: 2rem;
        height: 2rem;
        margin-right: 1rem;
      }
      .token-name {
        font-size: 16px;
        font-weight: 400;
      }
    }
    .wrap-token-value {
      display: flex;
      flex-direction: column;
      align-items: flex-end;
      .token-value {
        font-size: 18px;
        font-weight: 600;
      }
      .token-value-fiat {
        font-size: 14px;
        color: var(--gray900);
      }
    }
  }
}
</style>
