<template>
  <div>
    <button v-if="accounts.length === 0" @click="connectToMetamask()"> Connect to Metamask </button>
    <button v-else @click="disconnect()">Disconnect</button>
    <div>Account: {{ accounts[0] }} </div>
    <div>TokenA Balance: {{ tokenABalance }} TKA </div>
    <div>TokenB Balance: {{ tokenBBalance }} TKB </div>

    <div class="container">
      <h1>Token Swap</h1>
      
      <form @submit.prevent="createSwapOffer()">
        <select v-model="swapForm.swapType">
          <option :value="0">Token A for Token B</option>
          <option :value="1">Token B for Token A</option>
        </select>
        <div>Amount From: </div>
        <input v-model.number="swapForm.amountFrom"/>
        <div>Amount To: </div>
        <input v-model.number="swapForm.amountTo"/>

        <div>
          <button type="submit">Create Swap Offer</button>
        </div>
      </form>
      
      <div 
        v-for="(swapOffer, index) in swapOffers" 
        :key="index"
        class="card-swap">
        <div>{{ swapOffer }}</div>
        <div>{{ swapOffer.swapIndex }} Swap Offer by: {{ swapOffer.creator }} </div> 
        <div v-if="swapOffer.swapType === 0">
          {{ swapOffer.fromToken }} Token A for
          {{ swapOffer.toToken }} Token B
        </div>
        <div v-else>
          {{ swapOffer.fromToken }} Token B for
          {{ swapOffer.toToken }} Token A
        </div>
        <div>
          <button v-if="!swapOffer.isSwapped" @click="fulfillSwapOffer(swapOffer)">Swap</button>
        </div>
      </div>
    </div>
  </div>  
</template>
<script>
import { ethers, utils } from 'ethers'
import tokenAAbi from '../contract-abis/tokenA.json'
import tokenBAbi from '../contract-abis/tokenB.json'
import tokenSwapAbi from '../contract-abis/tokenSwap.json'


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
      tokenAContract: "0x320C911a36eAf7ecde9CF35fC38d92e7C2b25B5d",
      tokenBContract: "0x2D49dAd6e04ccD0a04291631837Fe5123d2cE004",
      tokenSwapContract: "0xaFd6CE11d3d928fBCF8B9415CEa57d3B32585c3f",
      message: '',
      swapForm: {
        swapType: 0,
        amountFrom: 0,
        amountTo: 0,
      },
      swapOffers: [],
    }
  },
  methods: {
    async fulfillSwapOffer(swapOffer){
      var signer = this.provider.getSigner(signer)
      var tokenSwapWithSigner = this.tokenSwap.connect(signer)

      var tokenTo 
      if(swapOffer.swapType === 0){
        tokenTo = this.tokenB
      } else {
        tokenTo = this.tokenA
      }

      console.log(tokenTo)
      var tokenToWithSigner = tokenTo.connect(signer)
      var approveTx = await tokenToWithSigner.approve(
        this.tokenSwap.address,
        swapOffer.amountTo
      )

      await approveTx.wait()

      var transaction = await tokenSwapWithSigner.swap(swapOffer.swapIndex)
      await transaction.wait()
      this.getSwapsList()
      this.getTokenBalances()
    },
    async getSwapsList(){
      var swapsLength = await this.tokenSwap.getSwapsLength()
      this.swapOffers = []

      for(var i=0; i<swapsLength; i++){
        var swap = await this.tokenSwap.swaps(i)
        var _amountFrom = utils.formatUnits(swap.amountFrom, 18)
        var _amountTo = utils.formatUnits(swap.amountTo, 18)
        var _swap = {
          swapIndex: i,
          swapType: swap.swapType,
          creator: swap.creator,
          swappedBy: swap.swappedBy,
          fromToken: _amountFrom,
          toToken: _amountTo,
          amountFrom: swap.amountFrom,
          amountTo: swap.amountTo,
          isSwapped: swap.isSwapped
        }
        this.swapOffers.push(_swap)
      }
    },
    async createSwapOffer(){
      var signer = this.provider.getSigner(signer)
      var tokenSwapWithSigner = this.tokenSwap.connect(signer)
      var amountFrom = utils.parseUnits(String(this.swapForm.amountFrom), 18)
      var amountTo = utils.parseUnits(String(this.swapForm.amountTo), 18)

      var tokenFrom
      if(this.swapForm.swapType === 0){
        tokenFrom = this.tokenA
      } else {
        tokenFrom = this.tokenB
      }

      var tokenFromWithSigner = tokenFrom.connect(signer)
      var approveTx = await tokenFromWithSigner.approve(
        this.tokenSwap.address,
        amountFrom
      )
      await approveTx.wait()

      var transaction = await tokenSwapWithSigner.createSwap(
        this.swapForm.swapType,
        amountFrom,
        amountTo
      )

      await transaction.wait()
      this.getSwapsList()
    },
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
      this.tokenSwap = new ethers.Contract(this.tokenSwapContract, tokenSwapAbi)
      this.tokenSwap  = this.tokenSwap.connect(this.provider)

      this.tokenA = new ethers.Contract(this.tokenAContract, tokenAAbi)
      this.tokenA  = this.tokenA.connect(this.provider)

      this.tokenB = new ethers.Contract(this.tokenBContract, tokenBAbi)
      this.tokenB  = this.tokenB.connect(this.provider)

      this.getTokenBalances()
      this.getSwapsList()
    },
    async getTokenBalances() {
      var _tokenABalance = await this.tokenA.balanceOf(this.accounts[0])
      this.tokenABalance = utils.formatUnits(_tokenABalance, 18)

      var _tokenBBalance = await this.tokenB.balanceOf(this.accounts[0])
      this.tokenBBalance = utils.formatUnits(_tokenBBalance, 18)
    },
  },
}
</script>
<style scoped>
.card-swap{
  border: 1px solid black;
  margin-bottom: 10px;
}
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