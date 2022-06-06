<template>
  <div>
    <h2>Staking contract: {{ txn }}</h2>
    <p><button v-on:click="connect">Connect</button></p>

    <div id="stake">
      <form v-on:submit.prevent="stakeContract">
        <p>Stake Amount: <input v-model="amount_" id="input"> <br><br>
          Stake Period: <input v-model="period_" id="input"></p><br>
        <button id="button">STAKE</button>
      </form>
    </div>
    <div id="unstake">
      <button v-on:click="getDetails">Get details</button>
      <p>Your balance is {{ balance_ }} tokens <br>
        {{ timeleft_ }}</p>
      <button v-on:click="unstakeContract">UNSTAKE</button>
    </div>
  </div>
</template>

<!-- <script setup>
</script> -->

<script setup>

import { ref } from "vue";

const { ethers } = require('ethers')
// const bigInt = require("big-integer");
const ABI_TOKEN = require('../abis/Token.json')
const ABI_STAKE = require('../abis/Staking.json')
const STAKE_ADDRESS = "0x420135F0dBd4c2a8C2dddF44a652e5d53DC3Ae98"
const TOKEN_ADDRESS = "0x319e8ff9de35074db065B62d475e5513c22d261c"
let isStaked = false
let StakingProcess = false

const balance_ = ref(0)
const timeleft_ = ref(0)

const amount_ = ref(0)
const period_ = ref(0)


let provider = new ethers.providers.Web3Provider(window.ethereum)
// await provider.send("eth_requestAccounts", [])
let signer = provider.getSigner()

let readOnlyStake = new ethers.Contract(STAKE_ADDRESS, ABI_STAKE, provider)
//let readOnlyToken = new ethers.Contract(TOKEN_ADDRESS, ABI_TOKEN, provider)

let stake = new ethers.Contract(STAKE_ADDRESS, ABI_STAKE, signer)
let stakeSigner = stake.connect(signer)

let token = new ethers.Contract(TOKEN_ADDRESS, ABI_TOKEN, signer)
let tokenSigner = token.connect(signer)

async function stakeContract() {
  if (amount_.value == 0 || period_.value == 0) {
    alert("Please, enter amount of tokens and period of stake")
    return
  }
  if (StakingProcess) {
    alert("Please, Wait for the end of the current process!")
    return
  }

  if (!isStaked) {
    StakingProcess = true
    const txApproveResponse = await tokenSigner.approve(STAKE_ADDRESS, amount_.value, { gasLimit: 300000 })
    console.log(amount_.value)
    const txApproveReceipt = await txApproveResponse.wait()
    console.log(txApproveReceipt.transactionHash)
    const txStakeResponse = await stakeSigner.stake(amount_.value, period_.value, { gasLimit: 300000 })
    console.log(period_.value)
    const txStakeReceipt = await txStakeResponse.wait()
    console.log(txStakeReceipt.transactionHash)
    isStaked = true
    StakingProcess = false
    return { transaction: txApproveReceipt.transactionHash }
  }
  else { alert("You already have a stake!") }
}

async function unstakeContract() {
  if (StakingProcess) {
    alert("Please, Wait for the end of the current process!")
    return
  }
  if (!(isStaked)) {
    alert("You don't have a stake")
    return
  }
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let time = ethers.BigNumber.from(Math.round(Date.now() / 1000))
  time = time.sub(stake[2])
  if (Number(time) < Number(stake[3])) {
    alert("Please, wait until the end of the stake period")
    return
  }
  StakingProcess = true
  const txResponse = await stakeSigner.unstake({ gasLimit: 300000 })
  const txReceipt = await txResponse.wait()
  console.log(txReceipt.transactionHash)
  isStaked = false
  StakingProcess = false
}

async function getBalance() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let balance = 0
  let time = ethers.BigNumber.from(Math.round(Date.now() / 1000))
  time = time.sub(stake[2])
  if (Number(time) < Number(stake[3])) { balance = (Number(stake[0]) + Number(stake[0]) * 10 * Number(Math.round(Date.now() / 1000) - Number(stake[2])) / 100) }
  else { balance = (Number(stake[0]) + Number(stake[0]) * 10 * Number(stake[3]) / 100) }
  return balance
}

async function getTimeleft() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let timeleft = 0
  let time = ethers.BigNumber.from(Math.round(Date.now() / 1000))
  time = time.sub(stake[2])
  if (Number(time) < Number(stake[3])) { timeleft = Math.abs(Number(time.sub(stake[3]))) }
  else {
    timeleft = "You can unstake!"
    return timeleft
  }
  timeleft = "You can unstake in" + timeleft + "seconds"
  return timeleft
}

async function getDetails() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  if (stake[4]) {
    balance_.value = await getBalance()
    timeleft_.value = await getTimeleft()
  }
  else {
    balance_.value = 0
    timeleft_.value = "You don't have a stake"
  }
}

async function connect() {
  // let provider = new ethers.providers.Web3Provider(window.ethereum)
  await provider.send("eth_requestAccounts", [])
  const stake = await readOnlyStake.stakes(signer.getAddress())
  isStaked = stake[4]

}

</script>

<style>
#app {
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: whitesmoke;
  margin-top: 60px;
  font-size: x-large;
  text-shadow: 15px;
  font-weight: 900;
}

body {
  background-image: url(./assets/background.jpg);
}

div#stake {
  height: 28%;
  width: 25%;
  padding: 10px;
  margin: 10px;
  position: absolute;
  top: 40%;
  left: 15%;
  border: 4px solid gainsboro;
  border-radius: 25px;
  background-color: blueviolet;
  line-height: 1.5;
}

div#unstake {
  height: 28%;
  width: 25%;
  padding: 10px;
  margin: 10px;
  position: absolute;
  top: 40%;
  left: 60%;
  border: 4px solid gainsboro;
  border-radius: 25px;
  background-color: blueviolet;
  line-height: 1.5;
}


button {
  margin: 8px;
  height: 50px;
  width: 160px;
  background-color: aquamarine;
  border: 2px, darkblue;
  border-radius: 20px;
  font-size: x-large;
  color: indigo;
}

input {
  height: 25px;
  border: 2px, darkcyan;
  border-radius: 20px;
  text-align: center;
  font-family: 'Times New Roman', Times, serif;
  font-size: large;
}


p {
  margin: 15px;
}
</style>
