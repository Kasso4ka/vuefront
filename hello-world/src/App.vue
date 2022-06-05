<template>
  <img @click="tasst" alt="Vue logo" src="./assets/logo.png">
  <div>
    <h2>Staking contract: {{ txn }}</h2>
    <button v-on:click="tasst">Connect</button>
    <form v-on:submit.prevent="stakeContract">
      <p>Stake Amount: <input v-model="amount_"></p>
      <p>Stake Period: <input v-model="period_"></p>
      <button>STAKE</button>
    </form>
    <div id="app">
      <button v-on:click="getDetails">Get details</button>
      <h3>{{ balance_ }}</h3>
      <h3>{{ timeleft_ }}</h3>
    </div>
    <p><button v-on:click="unstakeContract">UNSTAKE</button></p>
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
  if (amount_.value == 0 || period_.value == 0) { alert("Please, enter amount of tokens") }
  else {
    const txApproveResponse = await tokenSigner.approve(STAKE_ADDRESS, amount_.value, { gasLimit: 300000 })
    console.log(amount_.value)
    const txApproveReceipt = await txApproveResponse.wait()
    console.log(txApproveReceipt.transactionHash)
    const txStakeResponse = await stakeSigner.stake(amount_.value, period_.value, { gasLimit: 300000 })
    console.log(period_.value)
    const txStakeReceipt = await txStakeResponse.wait()
    console.log(txStakeReceipt.transactionHash)
    return { transaction: txApproveReceipt.transactionHash }
  }
}

async function unstakeContract(){
    const txResponse = await stakeSigner.unstake({ gasLimit: 300000 })
    const txReceipt = await txResponse.wait()
    console.log(txReceipt.transactionHash)
}

async function getBalance() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let balance = 0
  let time = ethers.BigNumber.from(Math.round(Date.now() / 1000))
  time = time.sub(stake[2])
  if (Number(time) < Number(stake[3])) { balance = (Number(stake[0]) + Number(stake[0]) * 10 * Number(Math.round(Date.now() / 1000) - Number(stake[2])) / 100) }
  else { balance = (Number(stake[0]) + Number(stake[0]) * 10 * Number(stake[3]) / 100) }
  return { balance: balance }
}

async function getTimeleft() {
  const stake = await readOnlyStake.stakes(signer.getAddress())
  let timeleft = 0
  let time = ethers.BigNumber.from(Math.round(Date.now() / 1000))
  time = time.sub(stake[2])
  if (Number(time) < Number(stake[3])) { timeleft = Math.abs(Number(time.sub(stake[3]))) }
  else { timeleft = "You can unstake!" }
  return { timeleft: timeleft }
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

async function tasst() {
  // let provider = new ethers.providers.Web3Provider(window.ethereum)
  await provider.send("eth_requestAccounts", [])
  console.log(await stakeContract())
  // const stake = await readOnlyStake.stakes(signer.getAddress())
  // // let signer = provider.getSigner()
  // console.log(await signer.getAddress())
  // // const stake = await readOnlyStake.stakes(signer.getAddress())
  // console.log(await getBalance())
  // console.log(Number(stake[3]))
  // let time = ethers.BigNumber.from(Math.round(Date.now() / 1000))
  // time = time.sub(stake[2])
  // console.log(Number(time))
  // console.log(Number(stake[2]))
  // console.log(Math.round(Date.now() / 1000))
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
