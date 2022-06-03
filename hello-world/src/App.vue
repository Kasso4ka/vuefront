<template>
  <img @click="tasst" alt="Vue logo" src="./assets/logo.png">
  <div>
    <h2>Staking contract: {{ txn }}</h2>
    <button v-on:click="tasst">Connect</button>
    <form v-on:submit.prevent="tasst">
      <p>Stake Amount: <input v-model="offerPrice"></p>
      <p>Stake Period: <input v-model="offerPrice"></p>
      <button>STAKE</button>
    </form>
    <div id="app">
      <button v-on:click="getBalance">Get details</button>
      <h3>Instance owner: {{ getBalance() }}</h3>
    </div>
    <p><button v-on:click="tasst">UNSTAKE</button></p>
  </div>
</template>

<script>

const { ethers } = require('ethers')
//const TokenArtifact = require('../abis/Token.json')
const ABI_STAKE = require('../abis/Staking.json')
const STAKE_ADDRESS = "0x420135F0dBd4c2a8C2dddF44a652e5d53DC3Ae98"
//const TOKEN_ADDRESS = ""

let provider = new ethers.providers.Web3Provider(window.ethereum)
let signer = provider.getSigner()

let readOnlyStake = new ethers.Contract(STAKE_ADDRESS, ABI_STAKE, provider)
//let readOnlyToken = new ethers.Contract(TOKEN_ADDRESS, ABI_TOKEN, provider)

//let stake = new ethers.Contract(STAKE_ADDRESS, ABI_STAKE, signer)
//let stakeSigner = stake.connect(signer)

//let token = new ethers.Contract(TOKEN_ADDRESS, ABI_TOKEN, signer)
//let tokenSigner = token.connect(signer)


export default {
  name: 'App',
  data() {
    return {
      instanceOwner: '',
      description: '',
      askingPrice: '',
      instanceBuyer: '',
      offerPrice: ''
    }
  },
  methods: {
    async details() {
      this.instanceOwner = (await this.$root.core.getInstanceOwner()).instanceOwner
      this.description = (await this.$root.core.getDescription()).description
      this.askingPrice = (await this.$root.core.getAskingPrice()).askingPrice
      this.instanceBuyer = (await this.$root.core.getInstanceBuyer()).instanceBuyer
      this.offerPrice = (await this.$root.core.getOfferPrice()).offerPrice
    }
  },
  mounted() {
  }
}


async function getBalance() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let balance = 0
  if ((Date.now() - stake[2]) < stake[3]) { balance = (Number(stake[0]) + Number(stake[0]) * 10 * Number(Date.now() - stake[2]) / 100) }
  else { balance = (Number(stake[0]) + Number(stake[0]) * 10 * Number(stake[3]) / 100) }
  return { balance: balance }
}
async function getTimeleft() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let timeleft = 0
  if ((Date.now() - stake[2]) < stake[3]) { timeleft = Number(stake[3]) - (Date.now() - stake[2]) }
  else { timeleft = "You can unstake!" }
  return { timeleft: timeleft }
}

async function tasst() {
  // let provider = new ethers.providers.Web3Provider(window.ethereum)
  await provider.send("eth_requestAccounts", [])
  // let signer = provider.getSigner()
  console.log(await signer.getAddress())
  const stake = await readOnlyStake.stakes(signer.getAddress())
  console.log(stake)
  console.log(await getBalance())
  console.log(await getTimeleft())
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

p {
  padding: 10px;
  margin: 10px;
}
</style>
