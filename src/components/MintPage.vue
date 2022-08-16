<template>
    <div style="height: 100%">
        <div id="header" class="py-2">
            <div class="row">
                <div class="col-12 col-md-6 ps-5 my-2 text-start account-details">
                    <span class="account" :class="display"> Account: {{accounts[0]}}</span> <br>
                    <span class="balance" :class="display">Balance: {{ balance }}</span>
                </div>
                <div 
                    v-if="accounts.length>0"
                    class="col-12 col-md-6 px-4 my-2 text-end">
                    <button 
                         class="btn-disconnect col-12 col-md-4 py-11" 
                        @click="disconnect">Disconnect</button> 
                    <br>
                </div>
            </div>
        </div>

        <div v-if="accounts.length === 0" style="min-height:80%">
            <div class="container mt-4 pt-4">
                <h1>MINTING PROJECT</h1>
                <p>To Start Please Connect to Metamask</p>
                <button 
                    v-if="accounts.length === 0" class="btn-connect px-4 py-2" 
                    @click="connectToMetamask">Connect to Metamask</button> 
            </div>
        </div>

        <div v-else style="min-height: 80%">
            <div class="row mt-3">
                <div class="col-xs-12">
                    <h1 class="uppercase">{{ title }}</h1>
                    <div class="div-number-candidates">Number of NFTs: {{ nfts.length }}</div>
                    <div v-if="message.length>0">{{ message }}</div>
                    <div v-if="errorMessage.length>0" class="div-error">Error: {{ errorMessage }}</div>
                </div>
            </div>
            <div class="container">
                <div class="row mb-4">
                    <div class="col-12 text-center">
                        <button class="btn btn-primary" @click="mintNft()">Mint</button>
                    </div>
                </div>
                <div class="row">
                    <div class="col-3 mb-2" v-for="(nft, index) in nfts" :key="index">
                        <div class="card card-nft">
                            <img :src="nft.image" class="img-top" />

                            <h4 class="card-title text-uppercase">{{ nft.name }}</h4>
                            <div v-for="(attr, index2) in nft.attributes" :Key="index2">
                                <strong>{{ attr.trait_type }}</strong> - {{ attr.value }}
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div id="footer" class="mt-4 p-3 text-uppercase">
            <div class="row">
                <div class="col-6 text-start">
                    <a href="https://github.com/thompatacsil" target="new"><i class="fa-brands fa-github"></i></a>
                </div>
                <div class="col-6 text-end">
                    All Rights Reserved © 2022
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { ethers, utils } from 'ethers'
import erc721Abi from '../contract-abis/erc721.json'

export default{
    data(){
        return{
            errorMessage: '',
            title: "NFT Minting Page",
            message: "",
            provider: null,
            accounts: [],
            balance: 0,
            nfts: [],
            nftContract: null,
            display: "display-none",
            contractAddress: "0x8e325910927084D135a052A2f14Cbb65288Ac659",
        }
    },
    methods: {
        async getMintedNfts(){
            var _nfts = []
            var eventFilter = this.nftContract.filters.Minted()
            var logs = await this.nftContract.queryFilter(eventFilter)

            for(var i=0; i < logs.length; i++){
                var tokenId = logs[i].args.tokenId
                var tokenUri = await this.nftContract.tokenURI(tokenId)

                var _response = await fetch(tokenUri)
                var tokenMetadata = await _response.json()

                tokenMetadata.tokenId = tokenId
                _nfts.push(tokenMetadata)
            }
            this.nfts = _nfts
        },
        async mintNft(){
            var signer = this.provider.getSigner()
            var nftContractWithSigner = this.nftContract.connect(signer)
            var transaction = await nftContractWithSigner.mint({
                value: utils.parseEther('1')
            })
            await transaction.wait()
            alert("Minted an NFT")
            this.getMintedNfts()
        },
        disconnect(){
            this.provider = null
            this.accounts = []
            this.balance = 0
            this.nftContract = null
            this.errorMessage = ""
            this.display = "display-none"
        },
        createContractInstance(){
            this.nftContract = new ethers.Contract(this.contractAddress, erc721Abi)
            this.nftContract = this.nftContract.connect(this.provider)
            
            this.getMintedNfts()
        },
        async getBalance(){
            var rawBalance = await this.provider.getBalance(this.accounts[0])
            this.balance = utils.formatEther(rawBalance)
        },
        async connectToMetamask(){
            this.provider = new ethers.providers.Web3Provider(window.ethereum)
            this.accounts = await this.provider.send('eth_requestAccounts', [])
            
            this.display = "";
            this.getBalance()
            this.createContractInstance()
        },
    }
}
</script>

<style scoped>
    img{
        height: 100%;
        width: auto;
    }
    .account-details{
        color: white;
        font-size: 12px;
    }
    #header{
        background-color: #051821;
    }
    .container{
        height: 100%!important;
    }
    #footer{
        width: 100%;
        background-color: #051821;
        color: white;
        font-size: 12px;
    }
    #footer i{
        font-size: 18px;
    }
    #footer a:link, #footer a:visited{
        color: white;
    }
    #footer a:hover{
        color: #ccc;
    }  
    #footer a:active{
        color: #4A5487;
    }    
    .div-error{
        color: red;
        margin-bottom: 10px;
    }
    .display-none{
        display: none;
    }
    .uppercase{
        text-transform: uppercase;
    }
    .text-large{
        font-size: 25px;
    }
    .btn-connect{
        background-color: #4A5487; 
        color: white; 
        border-radius: 25px; 
        border: none;
    }
    .btn-disconnect{
        background-color: #4A5487; 
        color: white; 
        border-radius: 25px; 
        border: none;
    }
    .btn-connect-metamask:hover{
        background-color: #05B58B;
    }
</style>