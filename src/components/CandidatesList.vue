<template>
    <div>
        <h1 class="uppercase">{{ title }}</h1>
        <div class="div-number-candidates">Number of Candidates: {{ candidatesLength }}</div>
        <div v-if="message.length>0">{{ message }}</div>
        <div v-if="errorMessage.length>0" class="div-error">Error: {{ errorMessage }}</div>
        <div class="div-account" :class="display"> Account: {{accounts[0]}}</div>
        <div class="div-balance" :class="display">Balance: {{ balance }}</div>
        <div class="div-btn-connect"> 
            <button v-if="accounts.length === 0" class="btn-connect" @click="connectToMetamask">Connect to Metamask</button> 
            <button v-else class="btn-disconnect" @click="disconnect">Disconnect</button> 
        </div>
        
        <div class="container">
            <div class="row">
                <div class="col-md-4" v-for="(candidate, index) in candidates" :key="index">
                    <!-- <div class="div-container"> -->
                    <div class="card div-candidate">
                        <img 
                            :class="card-image-top"
                            :src="candidate.image" 
                            alt=""
                        >
                        <div class="card-body">
                            <h5 class="card-title"> Candidate # {{ index + 1 }}</h5>
                            <p class="card-text">
                                <span class="text-large uppercase"> {{ candidate.name }} </span> <br>
                                <span> Vote Count: {{ candidate.voteCount }} </span> <br>
                                <button v-if="votingEnabled === true" class="btn-vote" @click="addVote(index)">Vote</button>
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { ethers, utils } from 'ethers'
import ballotAbi from '../contract-abis/ballot.json'

export default{
    data(){
        return{
            errorMessage: '',
            title: "Candidates",
            message: "",
            provider: null,
            accounts: [],
            balance: 0,
            contract: null,
            candidatesLength: 0,
            candidates: [],
            display: "display-none",
            votingEnabled: true,
            contractAddress: "0x14615F912841F65b21a7Ac985232042eb1480F2f",
        }
    },
    methods: {
        disconnect(){
            this.provider = null
            this.accounts = []
            this.balance = 0
            this.contract = null
            this.candidatesLength = 0
            this.candidates = []
            this.display = "display-none"
            this.errorMessage = ""
        },
        async getCandidates(){
            this.candidatesLength = await this.contract.getCandidatesLength()
            this.candidates = []
            for(var i=0; i< this.candidatesLength; i++){
                var candidate = await this.contract.candidates(i)
                var _candidate = {
                    name: candidate.name,
                    voteCount: candidate.voteCount,
                    image: "https://avatars.dicebear.com/api/pixel-art/"+candidate.name+".svg"
                }
                this.candidates.push(_candidate)
            }
        },
        
        async getVotingState(){
            this.votingEnabled = await this.contract.votingEnabled()
            // this.votingEnabled = true
            if(this.votingEnabled === false){
                this.message = "Voting has been disabled by the Chairperson"
            } else {
                this.message = ""
            }
        },
        createContractInstance(){
            this.contract = new ethers.Contract(this.contractAddress, ballotAbi)
            this.contract = this.contract.connect(this.provider)
            
            this.getVotingState()
            this.getCandidates()
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
        async addVote(candidateIndex){
            var signer = this.provider.getSigner()
            var contractWithSigner = this.contract.connect(signer)

            try{
                var transaction = await contractWithSigner.vote(candidateIndex)
                await transaction.wait()
                this.errorMessage = ""
                this.getCandidates()
            }catch(error){
                this.errorMessage = error.data.message

                var message = this.errorMessage.split("revert")
                this.errorMessage = message[1]
            }
        }
    }
}
</script>

<style scoped>
    .div-candidate{
        margin-bottom: 10px;
    }
    .div-candidate-number{
        margin-bottom: 5px;
    }
    .div-error{
        color: red;
        margin-bottom: 10px;
    }
    .div-candidate img{
        width: auto;
        height: 100px;
    }
    .display-none{
        display: none;
    }
    .uppercase{
        text-transform: uppercase;
    }
    .div-number-candidates{
        transform: translateY(-10px);
    }
    .btn-vote{
        padding: 10px 10px; 
        background-color: #0D25A8; 
        color: white; 
        border-radius: 5px; 
        border: none;
        margin-top: 10px;
    }
    .btn-vote:hover{
        background-color: #05B58B;
    }
    .btn-vote:active{
        background-color: #156C94;
    }
    .text-large{
        font-size: 25px;
    }
    .div-btn-connect{
        position: absolute;
        top: 10px;
        right: 10px;
    }
    .btn-connect{
        padding: 10px 10px; 
        background-color: #156C94; 
        color: white; 
        border-radius: 5px; 
        border: none;
        margin-bottom: 20px;
        max-width: 250px;
        overflow: hidden;
    }
    .btn-disconnect{
        padding: 10px 10px; 
        background-color: #942215; 
        color: white; 
        border-radius: 5px; 
        border: none;
        margin-bottom: 20px;
    }
    .btn-connect-metamask:hover{
        background-color: #05B58B;
    }

    .div-account{
        position: absolute;
        top: 60px;
        right: 10px;
        font-size: 12px;
    }
    .div-balance{
        position: absolute;
        top: 80px;
        right: 10px;
        font-size: 12px;
    }
</style>