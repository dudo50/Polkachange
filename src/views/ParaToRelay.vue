<template>
  <div id="app">
    <div class="box" style="margin-top: 25%;  font-family: 'Anybody', cursive;">
     You are logged in as {{$store.state.account}}.
    </div>
    <b-field class="textt"  label-position="inside" label="Select origin parachain">
      <b-select expanded v-model="key" placeholder="Select parachain 1" required>
        <option v-for="(item) in items" :key="item">{{item}}</option>
      </b-select>
    </b-field>
    <b-field class="textt" label-position="inside" label="Input recipient address">
        <b-input expanded @input.native="addrs($event)" v-model="addr"></b-input>
    </b-field>
     <b-field class="textt" label-position="inside" label="Select currency">
      <b-select expanded v-model="currency" @input.native="asignCur($event)" placeholder="Select currency" required>
         <option v-for="(currency) in currencies" :key="currency">{{currency}}</option>
      </b-select>
    </b-field>
    <b-field class="textt" label-position="inside" label="Input currency amount">
        <b-input expanded @input.native="unit($event)" v-model="amount"></b-input>
    </b-field>
    <b-button class="buttonn" expanded type="is-primary" @click="sendXCM($store.state.account)">Send transaction</b-button>
  </div>
</template>
<script lang="ts">
  import { Keyring } from '@polkadot/api'
  import { ApiPromise, WsProvider } from '@polkadot/api'
  import { defineComponent } from '@vue/composition-api'
  import { decodeAddress } from '@polkadot/util-crypto'
  import '@polkadot/api-augment';

  export default defineComponent({
  
  data() {
    return {
        items: [] as Array<string>,
        key: "" as string,
        addr: "" as string,
        amount: 0 as number,
        currency: "" as string,
        currencies: [] as Array<string>,
        accounts: [] as Array<any>,
        recipient: "" as string,
      };
    },

  mounted: async function () {
    const keyring = new Keyring({ type: 'sr25519' });
    const wsProvider = new WsProvider('ws://127.0.0.1:9944');
    const api = await ApiPromise.create({ provider: wsProvider });
    const bob = keyring.addFromUri('//Alice', { name: 'Alice default' });
    const parachain = await api.query.paras.parachains()
    const queryPara = JSON.stringify(parachain)
    const newParas = queryPara.split('[').join(',').split(']').join(',').split(',')
    const results = newParas.filter(element => {return element !== "";});
    const extractedParas = results.map((i) => Number(i));
    for (let i=0;extractedParas.length>i; i++)
    {
      if (extractedParas[i] == 2090)
      this.items.push("Basilisk")
      else if(extractedParas[i]== 2000)
      this.items.push("Karura")
      else if(extractedParas[i] == 1000)
      this.items.push("Moonbeam")
    }

    this.currencies.push("UNIT")
  },

  methods: {
    async address(value: any){
      this.recipient=value.target.value
    },
    async addrs(value: any){
      this.addr=value.target.value
    },
    async asignCur(value: any){
      this.currency=value.target.value
    },
    async unit(value: any){
      this.amount=value.target.value
    },
    async sendXCM(address: string) {
      if(this.key == "") 
      {
        this.$notify({ title: 'Error', text: 'You need to select you wish to transfer from first.', type: 'error', duration: 3000,speed: 100})
      }
      else 
      {
        if(this.addr=="")
        {
          this.$notify({ title: 'Error', text: 'You need to select recipient first.', type: 'error', duration: 3000,speed: 100})
        }
        else{
          if(this.amount<1000000000000)
          {
           this.$notify({ title: 'Error', text: 'Specified amount is less than required {1000000000000}.', type: 'error', duration: 3000,speed: 100})
          }
          else{         
            var account = "//"+address         
            const keyring = new Keyring({ type: 'sr25519' });
            if(this.key == "Basilisk"){
              const wsProvider = new WsProvider('ws://127.0.0.1:9989');
              const api = await ApiPromise.create({ provider: wsProvider });
              const query = api.tx.xTokens.transfer(3,this.amount,{V1: {parents:1, interior: { X1: { AccountId32: { network: "any", id: api.createType('AccountId32', decodeAddress(this.addr)).toHex()}}}}}, 4600000000).signAndSend(keyring.createFromUri(account), (result) => { console.log(result) })
              this.$notify({ text: 'Your transfer is now processsing, refresh this page in few seconds to see changes.', duration: 10000,speed: 100})
            }
            else if(this.key == "Karura"){
              const wsProvider = new WsProvider('ws://127.0.0.1:9988');
              const api = await ApiPromise.create({ provider: wsProvider });
              const query = api.tx.xTokens.transfer({Token: "KSM"},this.amount,{V1: {parents:1, interior: { X1: { AccountId32: { network: "any", id: api.createType('AccountId32', decodeAddress(this.addr)).toHex()}}}}}, 4600000000).signAndSend(keyring.createFromUri(account), (result) => { console.log(result) })
              this.$notify({ text: 'Your transfer is now processsing, refresh this page in few seconds to see changes.', duration: 10000,speed: 100})

            }                      
            else if (this.key == "Moonbeam"){
              const wsProvider = new WsProvider('ws://127.0.0.1:9999');
              const api = await ApiPromise.create({ provider: wsProvider });
              const query = api.tx.polkadotXcm.reserveTransferAssets({V1: { parents:1, interior:"Here"}},
              {V1:{parents:1,interior:{X1:{AccountId32: {network: "Any", key: this.addr}}}}},
              {V1: [{id: {Concrete: {parents:1, interior: "Here"}}, fun:{ Fungible: this.amount}}]},0).signAndSend("0xf24FF3a9CF04c71Dbc94D0b566f7A27B94566cac", (result) => { console.log(result) })
              this.$notify({ text: 'Your transfer is now processsing, refresh this page in few seconds to see changes.', duration: 10000,speed: 100})
            }  
          }
        }
      }
    }
  }
})
</script>
<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Anybody:wght@300&family=BIZ+UDGothic&family=Pacifico&display=swap");

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: white;
  margin-top: 20px;
  margin-left: 20%;
  margin-right: 20%;
}
    select {
        width: 150px;
        margin: 10px;
    }
    select:focus {
        min-width: 150px;
        width: auto;
    }
.textt{
  color: black;
  font-family: "Anybody", cursive;
  font-size: 30px;
  margin-bottom: 20px;
}
</style>
