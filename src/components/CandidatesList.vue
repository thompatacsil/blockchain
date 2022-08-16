<template>
    <div style="height: 100vh">
        <div id="header" class="py-2">
            <div class="row">
                <div class="col-6 col-md-6 ps-5 my-2 text-start account-details text-truncate">
                    <span class="account" :class="display"> Account: {{accounts[0]}}</span> <br>
                    <span class="balance" :class="display">Balance: {{ balance }}</span>
                </div>
                <div class="col-6 col-md-6 px-4 my-2 pt-2 text-end">
                    <!-- <button 
                        v-if="accounts.length>0" class="btn-disconnect col-12 col-md-4 py-1" 
                        @click="disconnect">Disconnect</button>  -->
                    <div class="row">
                        <div class="col-11">
                            <div class="form-check form-switch" style="margin-left: 96%">
                                <!-- <label class="form-check-label" for="flexSwitchCheckDefault">Enable Voting</label> -->
                                <input 
                                v-if="isChairperson"
                                class="form-check-input" type="checkbox" id="flexSwitchCheckDefault" 
                                data-bs-toggle="tooltip" data-bs-placement="bottom" title="Enable Voting State">
                            </div>
                        </div>
                        <div class="col-1">
                            <a href="#" 
                                v-if="accounts.length>0" 
                                @click="disconnect" 
                                class="me-4"
                                data-bs-toggle="tooltip" data-bs-placement="bottom" title="Disconnect">
                                <i class="fa-solid fa-arrow-right-from-bracket" style="font-size: 18px;"></i>
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="content" v-if="accounts.length === 0">
            <div class="container mt-4 pt-4">
                <h1>BALLOT PROJECT</h1>
                <p>To Start Please Connect to Metamask</p>
                <button 
                    v-if="accounts.length === 0" class="btn-connect px-4 py-2" 
                    @click="connectToMetamask">Connect to Metamask</button> 
            </div>
        </div>

        <div class="content" v-else>
            <div class="row mt-4 pt-4">
                <div class="col-xs-12">
                    <h1 class="uppercase">{{ title }}</h1>
                    <div class="div-number-candidates">Number of Candidates: {{ candidatesLength }}</div>
                    <div v-if="message.length>0" class="text-danger">{{ message }}</div>
                    <div v-if="errorMessage.length>0" class="div-error">Error: {{ errorMessage }}</div>
                    {{ chairperson }}
                </div>
            </div>
            <div class="container">
                <div class="row">
                    <div class="col-md-4" v-for="(candidate, index) in candidates" :key="index">
                        <div class="card div-candidate">
                            <img 
                                class="card-image-top"
                                :src="candidate.image" 
                                alt=""
                            >
                            <div class="card-body">
                                <h5 class="card-title"> Candidate # {{ index + 1 }}</h5>
                                <p class="card-text">
                                    <span class="text-large uppercase"> {{ candidate.name }} </span> <br>
                                    <span> Vote Count: {{ candidate.voteCount }} </span>
                                </p>
                                <button v-if="votingEnabled === true" class="btn-vote" @click="addVote(index)">Vote</button>
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
            winner: "",
            display: "display-none",
            votingEnabled: true,
            contractAddress: "0x5431a6b5582bc8bD497732638E502589d6D648BD",
            chairperson: "",
            isChairperson: false,
            contractAddress: "0xd21a001ec48C9D44CB70cc6129420dB107e50176",
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
            this.winner = ""
        },
        async getChairperson(){
            this.chairperson = "";
            this.chairperson = await this.contract.chairperson()

            if(this.chairperson == this.accounts[0]){
                this.isChairperson = true;
            }
            console.log(this.chairperson)
            console.log(this.accounts[0])
            console.log(this.isChairperson)
        },
        async getWinner(){
            this.winner = ""
            this.winner = await this.contract.getWinner()
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
        async setVotingState(votingState){
            var signer = this.provider.getSigner()
            var contractWithSigner = this.contract.connect(signer)
            try{
                var transaction = await contractWithSigner.setVotingState(votingState)
                await transaction.wait()
                this.errorMessage = ""
                this.getCandidates()
            }catch(error){
                this.errorMessage = error.data.message

                var message = this.errorMessage.split("revert")
                this.errorMessage = message[1]
            }
        },
        async getVotingState(){
            this.votingEnabled = await this.contract.votingEnabled()
            if(this.votingEnabled === false){
                this.message = "Voting has been disabled by the Chairperson"
            } else {
                this.message = ""
            }
        },
        createContractInstance(){
            this.contract = new ethers.Contract(this.contractAddress, ballotAbi)
            this.contract = this.contract.connect(this.provider)
            
            this.getChairperson()
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
    .content{
        /* height: 82%; */
        min-height: 83%;
    }
    .account-details{
        color: white;
        font-size: 12px;
    }
    #header{
        background-color: #051821;
        /* height: 7%; */
    }
    .container{
        height: 100%!important;
    }
    #footer{
        /* height: 5%; */
        width: 100%;
        background-color: #051821;
        color: white;
        font-size: 12px;
    }
    #footer i{
        font-size: 18px;
    }
    #footer a:link, #footer a:visited, 
    #header a:link, #header a:visited{
        color: white;
    }
    #footer a:hover,
    #header a:hover{
        color: #ccc;
    }  
    #footer a:active,
    #header a:active{
        color: #4A5487;
    }    
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
        padding: 5px 10px; 
        background-color: #041154; 
        color: white; 
        border-radius: 5px; 
        border: none;
    }
    .btn-vote:hover{
        background-color: #4A5487;
    }
    .btn-vote:active{
        background-color: #051821;
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