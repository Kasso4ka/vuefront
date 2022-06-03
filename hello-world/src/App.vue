<template>
  <img @click="tasst" alt="Vue logo" src="./assets/logo.png">
</template>

<script setup>

const { ethers } = require('ethers')
//const TokenArtifact = require('../abis/Token.json')
const ABI_STAKE = require('../abis/Staking.json')
const STAKE_ADDRESS = "0x420135F0dBd4c2a8C2dddF44a652e5d53DC3Ae98"
//const TOKEN_ADDRESS = ""

/*const ABI_STAKE = [
  'function stake(uint256 amount, uint256 period) public',
  'function unstake() public',
  'function myStake(address user)	external view returns (uint256 amount, uint256 timestamp, uint256 period)',
  'function stakes[address user].amount view returns (uint256 amount)'
]*/

/*const ABI_TOKEN = [
  'function approve(address spender, uint256 amount) external returns (bool)',
  'function balanceOf(address account) external view returns (uint256)',
]*/

let provider = new ethers.providers.Web3Provider(window.ethereum)
let signer = provider.getSigner()

let readOnlyStake = new ethers.Contract(STAKE_ADDRESS, ABI_STAKE, provider)
//let readOnlyToken = new ethers.Contract(TOKEN_ADDRESS, ABI_TOKEN, provider)

//let stake = new ethers.Contract(STAKE_ADDRESS, ABI_STAKE, signer)
//let stakeSigner = stake.connect(signer)

//let token = new ethers.Contract(TOKEN_ADDRESS, ABI_TOKEN, signer)
//let tokenSigner = token.connect(signer)

async function getBalance() {
}
async function getTimeleft() {
  const timeleft = await (readOnlyStake.myStake(signer.getAddress()).period -
    (readOnlyStake.myStake(signer.getAddress()).timestamp - Date.now()))
  return { timeleft: timeleft }
}

async function tasst() {
  // let provider = new ethers.providers.Web3Provider(window.ethereum)
  await provider.send("eth_requestAccounts", [])
  // let signer = provider.getSigner()
  console.log(await signer.getAddress())
  const returnn = readOnlyStake.stakes(signer.getAddress())
  console.log(returnn)
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
</style>
