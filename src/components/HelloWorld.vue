<script setup>
import { ref } from 'vue'
import {WalletHandler, FileIo, FileUploadHandler, StorageHandler, getFileTreeData} from 'jackal.js'

defineProps({
  msg: String,
})

const tSignerChain = 'lupulella-2'
const testnet = {
  signerChain: tSignerChain,
  enabledChains: [tSignerChain],
  queryAddr: 'https://testnet-grpc.jackalprotocol.com',
  txAddr: 'https://testnet-rpc.jackalprotocol.com',
  chainConfig: {
    chainId: tSignerChain,
    chainName: 'Jackal Testnet II',
    rpc: 'https://testnet-rpc.jackalprotocol.com',
    rest: 'https://testnet-api.jackalprotocol.com',
    bip44: {
      coinType: 118
    },
    coinType: 118,
    stakeCurrency: {
      coinDenom: 'JKL',
      coinMinimalDenom: 'ujkl',
      coinDecimals: 6
    },
    bech32Config: {
      bech32PrefixAccAddr: 'jkl',
      bech32PrefixAccPub: 'jklpub',
      bech32PrefixValAddr: 'jklvaloper',
      bech32PrefixValPub: 'jklvaloperpub',
      bech32PrefixConsAddr: 'jklvalcons',
      bech32PrefixConsPub: 'jklvalconspub'
    },
    currencies: [
      {
        coinDenom: 'JKL',
        coinMinimalDenom: 'ujkl',
        coinDecimals: 6
      }
    ],
    feeCurrencies: [
      {
        coinDenom: 'JKL',
        coinMinimalDenom: 'ujkl',
        coinDecimals: 6,
        gasPriceStep: {
          low: 0.002,
          average: 0.002,
          high: 0.02
        }
      }
    ],
    features: []
  }
}

const mSignerChain = 'jackal-1'
const mainnet = {
  signerChain: mSignerChain,
  enabledChains: [mSignerChain],
  queryAddr: 'https://grpc.jackalprotocol.com',
  txAddr: 'https://rpc.jackalprotocol.com',
  chainConfig: { // mainnet chain config
      chainId: mSignerChain,
      chainName: 'Jackal',
      rpc: 'https://rpc.jackalprotocol.com',
      rest: 'https://api.jackalprotocol.com',
      bip44: {
        coinType: 118
      },
      coinType: 118,
      stakeCurrency: {
        coinDenom: 'JKL',
        coinMinimalDenom: 'ujkl',
        coinDecimals: 6
      },
      bech32Config: {
        bech32PrefixAccAddr: 'jkl',
        bech32PrefixAccPub: 'jklpub',
        bech32PrefixValAddr: 'jklvaloper',
        bech32PrefixValPub: 'jklvaloperpub',
        bech32PrefixConsAddr: 'jklvalcons',
        bech32PrefixConsPub: 'jklvalconspub'
      },
      currencies: [
        {
          coinDenom: 'JKL',
          coinMinimalDenom: 'ujkl',
          coinDecimals: 6
        }
      ],
      feeCurrencies: [
        {
          coinDenom: 'JKL',
          coinMinimalDenom: 'ujkl',
          coinDecimals: 6,
          gasPriceStep: {
            low: 0.002,
            average: 0.002,
            high: 0.02
          }
        }
      ],
      features: []
  }
}

const count = ref(0)

let fid = ref("waiting for upload...")

let wallet = ref({})

let fileIo = null

const connectWallet = async function() {

  const walletConfig = {
    selectedWallet: 'keplr',
    signerChain: mainnet.signerChain,
    enabledChains: mainnet.enabledChains,
    queryAddr: mainnet.queryAddr,
    txAddr: mainnet.txAddr,
    chainConfig: mainnet.chainConfig
  }

  // Hooking up the wallet to your app
  wallet.value = await WalletHandler.trackWallet(walletConfig)

  fileIo = await FileIo.trackIo(wallet.value, '1.0.9')

  const listOfFolders = ["publish-demo"] 
  // you can create as many folders as you would like this way


  // after the first time, this code can be used instead. this will only create new folders if they don't already exist
  const newFolderCount = await fileIo.verifyFoldersExist(listOfFolders)
  console.log(newFolderCount)
}

const uploadFile = async function (file) {

  const parentFolderPath = "s/publish-demo" // replace this with your own path
  const fileName = file.name // replace this with your own file name
  const handler = await FileUploadHandler.trackFile(file, parentFolderPath)

  const parent = await fileIo.downloadFolder(parentFolderPath)
  console.log(parent)
  const uploadList = {}
  uploadList[fileName] =  {
    data: null,
    exists: false, 
    handler: handler,
    key: fileName,
    uploadable: await handler.getForPublicUpload()
  }
  
  const details = await fileIo.staggeredUploadFiles(uploadList, parent, {counter: 0, complete: 0})
  console.log(details)

  const f = await getFileTreeData("s/publish-demo/" + file.name, wallet.value.getJackalAddress(), wallet.value.getQueryHandler())

  const fidList = JSON.parse(f.value.files.contents)
  const newFid = fidList.fids[0]
  console.log(newFid)
  fid.value = newFid
}

const onChange = function (e) {
  const files = e.target.files
  console.log(files)

  
  uploadFile(files[0])
}

const goToFID = function () {
 window.open(`https://jackal.link/f/${fid.value}`, '_blank')
}


</script>


<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <button type="button" @click="connectWallet()">{{wallet.value ? "Connected" : "Connect Wallet"}}</button>
        <h3>Upload images</h3>
        <div class="dropbox">
        <input type="file" class="input-file" @change="onChange">
        <p>{{fid}}</p>
        <button v-if="fid.startsWith('jklf')" @click="goToFID">Check it out!</button>
      </div>
  </div>

  
</template>

<script>

 
</script>


<style scoped>

</style>
