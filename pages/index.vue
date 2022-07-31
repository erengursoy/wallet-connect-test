<template>
 <div class="inline flex justify-center">
 
    <a @click="connectWallet()" class="inline-block p-7 border-1 border-blue-800 bg-gray-100"  href="">connect wallet</a>
  
 </div>
</template>

<script>

import Web3 from "web3";
import Web3Modal from "web3modal";
import WalletConnectProvider from "@walletconnect/web3-provider";
import {mapGetters} from 'vuex'


export default {
  name: "app-header",
  mounted() {
    const self = this;
    this.$nextTick(function () {

      if (self.$route.hash !== '') {
        setTimeout(() => self.scrollTO(self.$route.hash), 500)
        setTimeout(() => {
          window.history.replaceState({}, '', '/');
        }, 500)
      }

      self.connectWallet();

      // window.addEventListener("scroll", function () {
      //   const nav = document.querySelector('nav')
      //   if (document.documentElement.scrollTop >= 30) {
      //     nav.style.backgroundColor = '#e5e3e3'
      //   } else {
      //     nav.style.backgroundColor = 'transparent'
      //   }
      // });

    })
  },
  computed: mapGetters({
    walletConnection: 'wallet/walletConnection',
  }),
  methods: {
    async connectWallet() {
      const self = this;

      try {

        const providerOptions = {
          walletconnect: {
            package: WalletConnectProvider,
            options: {
              infuraId: process.env.INFURA_ID
            }
          },
        };

        const web3Modal = new Web3Modal({
          network: process.env.NETWORK_SLUG, // optional
          cacheProvider: true, // optional
          disableInjectedProvider: false,
          providerOptions // required
        });


        const provider = await web3Modal.connect();


        const web3 = new Web3(provider);


        const acc = await web3.eth.getAccounts();


        if (acc.length > 0) {

          const chainId = await web3.eth.getChainId();

          if (chainId !== parseInt(process.env.NETWORK_CHAINID)) {

            const storeObj = {
              connectionReady: true,
              isConnected: true,
              connectionStatus: 'falseNetwork',
              connectionMessage: 'please switch to ' + process.env.NETWORK_NAME,
              statusInfo: {
                networkToBind: process.env.NETWORK_NAME
              },
              address: acc[0],
            }
            await self.$store.dispatch('wallet/setConnection', storeObj)

          } else {
            self.$axios({method: 'get', url: process.env.CONTRACT_JSONURL, baseURL: '/'})
              .then(async (res) => {
                const nftAbi = res.data.abi
                const nft = new web3.eth.Contract(nftAbi, process.env.CONTRACT_ADDRESS);
                const address = acc[0]

                const storeObj = {
                  connectionReady: true,
                  isConnected: true,
                  connectionStatus: 'connected',
                  connectionMessage: 'Successfully connected to ' + address,
                  statusInfo: {},
                  address: acc[0],
                  contractAddress: process.env.CONTRACT_ADDRESS,
                  etherScan: process.env.ETHERSCAN,
                  nftMethods: nft.methods
                }

                self.$axios.defaults.headers.common['X-Wallet'] = address;
                await self.$store.dispatch('wallet/setConnection', storeObj)

                self.$forceUpdate()


              })
          }


          self.isConnected = true;

          self.$forceUpdate()
          // chainId
          // process..env.DEV_NETWORK_CHAINID


        }

        // Subscribe to accounts change
        provider.on("accountsChanged", (accounts) => {
          // console.log(accounts);
          if (accounts.length > 0) localStorage.removeItem('WEB3_CONNECT_CACHED_PROVIDER');
          window.location.reload()
        });

        // Subscribe to chainId change
        provider.on("chainChanged", (chainId) => {
          // console.log(chainId);
          window.location.reload()
        });

        // Subscribe to provider connection
        provider.on("connect", (info) => {
          // console.log(info);
          window.location.reload()
        });

        // Subscribe to provider disconnection
        provider.on("disconnect", (error) => {
          localStorage.removeItem('WEB3_CONNECT_CACHED_PROVIDER')
          window.location.reload()
          // console.log(error);
        });

      } catch (e) {
        console.log('error', e)
      }


    },
  }
}
</script>