<template>
  <div>
    <button v-if="accounts.length === 0" @click="connectToMetamask()"> Connect to Metamask </button>
    <button v-else @click="disconnect()">Disconnect</button>
    <div>Account: {{ accounts[0] }} </div>
    <div>TokenA Balance: {{ tokenABalance }} TKA </div>
    <div>TokenB Balance: {{ tokenBBalance }} TKB </div>

    <div class="container">
      <h1>Token Swap</h1>
    </div>
  </div>  
</template>
<script>
import { ethers } from 'ethers'

export default {
  data() {
    return {
      provider: null,
      accounts: [],
      tokenSwap: null,
      tokenA: null,
      tokenB: null,
      tokenABalance: 0,
      tokenBBalance: 0,
    }
  },
  methods: {
    async connectToMetamask() {
      this.provider = new ethers.providers.Web3Provider(window.ethereum)
      this.accounts = await this.provider.send('eth_requestAccounts', [])

      this.createContractInstances()
    },
    disconnect() {
      this.provider = null
      this.accounts = []

      this.tokenSwap = null
      this.tokenA = null
      this.tokenB = null

      this.tokenABalance = 0
      this.tokenBBalance = 0
    },
    createContractInstances() {
      this.tokenSwap = new ethers.Contract('', )
      this.tokenSwap  = this.tokenSwap.connect(this.provider)

      this.tokenA = new ethers.Contract('', )
      this.tokenA  = this.tokenA.connect(this.provider)

      this.tokenB = new ethers.Contract('', )
      this.tokenB  = this.tokenB.connect(this.provider)

      this.getTokenBalances()
    },
    async getTokenBalances() {
      
    },
  },
}
</script>
<style scoped>
button {
  color: white;
  background-color: green;
  padding: 10px 20px;
  margin: 5px;
  border: none;
  border-radius:  5px;
}
button:active {
  background-color: darkgreen;
}

</style>