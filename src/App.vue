<template>
  <div>
    MATIC Balance: {{ maticBalance }}
    <br />

    <div v-for="(balance, token) in tokenBalances" :key="token">
      {{ token }} Balance: {{ balance }}
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'
import Web3 from 'web3'

export default defineComponent({
  setup() {
    const maticBalance = ref('Loading...')
    const tokenBalances = ref({})

    onMounted(async () => {
      const web3 = new Web3('https://rpc-mainnet.maticvigil.com')
      const address = '0xc834bD2C217835E770b3Ba3d6c1D38eD45d5c291'
      const tokens = {
        stMATIC: '0x6d80113e533a2C0fe82EaBD35f1875DcEA89Ea97'
        // Add other tokens here
      }

      const erc20Abi = [
        {
          constant: true,
          inputs: [],
          name: 'name',
          outputs: [{ name: '', type: 'string' }],
          payable: false,
          type: 'function'
        },
        {
          constant: true,
          inputs: [],
          name: 'totalSupply',
          outputs: [{ name: '', type: 'uint256' }],
          payable: false,
          type: 'function'
        },
        {
          constant: true,
          inputs: [],
          name: 'decimals',
          outputs: [{ name: '', type: 'uint8' }],
          payable: false,
          type: 'function'
        },
        {
          constant: true,
          inputs: [{ name: '_owner', type: 'address' }],
          name: 'balanceOf',
          outputs: [{ name: 'balance', type: 'uint256' }],
          payable: false,
          type: 'function'
        },
        {
          constant: false,
          inputs: [
            { name: '_to', type: 'address' },
            { name: '_value', type: 'uint256' }
          ],
          name: 'transfer',
          outputs: [],
          payable: false,
          type: 'function'
        },
        {
          constant: true,
          inputs: [],
          name: 'symbol',
          outputs: [{ name: '', type: 'string' }],
          payable: false,
          type: 'function'
        },
        {
          constant: false,
          inputs: [
            { name: '_from', type: 'address' },
            { name: '_to', type: 'address' },
            { name: '_value', type: 'uint256' }
          ],
          name: 'transferFrom',
          outputs: [],
          payable: false,
          type: 'function'
        },
        {
          constant: false,
          inputs: [
            { name: '_spender', type: 'address' },
            { name: '_value', type: 'uint256' }
          ],
          name: 'approve',
          outputs: [],
          payable: false,
          type: 'function'
        },
        {
          constant: true,
          inputs: [
            { name: '_owner', type: 'address' },
            { name: '_spender', type: 'address' }
          ],
          name: 'allowance',
          outputs: [{ name: 'remaining', type: 'uint256' }],
          payable: false,
          type: 'function'
        }
      ]

      const balanceWei = await web3.eth.getBalance(address)
      maticBalance.value = web3.utils.fromWei(balanceWei, 'ether')

      for (const token in tokens) {
        const contract = new web3.eth.Contract(erc20Abi, tokens[token])
        const tokenName = await contract.methods.name().call()
        const balanceWei = await contract.methods.balanceOf(address).call()
        const balance = web3.utils.fromWei(balanceWei, 'ether')
        if (parseFloat(balance) > 0) {
          tokenBalances.value[tokenName] = balance
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
