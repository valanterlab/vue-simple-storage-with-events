<template>
  <div>
    <div class="jumbotron">
      <h1>{{ title }}</h1>
      <p>{{ msg }}</p>
      <p>
        <a class="btn btn-lg btn-primary" href="https://github.com/valanterlab/vuesimplestorage-withevents" role="button">View on GitHub</a>
      </p>
    </div>

    <div class="container">
      <div class="row">
        <div class="col-md-5 text-left">
          <h3>SimpleStorage.sol</h3>
          <pre>
{{ contractSource }}
          </pre>
          <h3>Abi</h3>
          <pre>[{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]</pre>
          <h3>Bytecode</h3>
          <pre>0x6060604052341561000f57600080fd5b60d38061001d6000396000f3006060604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c14606e575b600080fd5b3415605857600080fd5b606c60048080359060200190919050506094565b005b3415607857600080fd5b607e609e565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a7230582030036eed4617b76ee4550080d2fced7bfbc3ddba9d7b7212901e539bd92c6f5a0029</pre>
        </div>
        <div class="col-md-7 text-left">
          <h3>Get value from the storage</h3>
          <p>
            Set a value to call the event and get the current storage value.
          </p>
          <div class="row" v-if="retreivedValue">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert">{{ retreivedValue }}</div>
            </div>
          </div>
          <div class="row" v-if="!retreivedValue">
            <div class="col-md-12 text-center">
              <div class="alert alert-info" role="alert">Waiting</div>
            </div>
          </div>
          <hr />
          <h3>Node informations</h3>
          <p>
            The Euthereum node url to communicate with. It should be WebSocket (ws://...) url.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="nodeUrl" class="col-md-4 control-label text-right">Node url</label>
              <div class="col-md-8">
                <input
                  type="text"
                  id="nodeUrl"
                  class="form-control"
                  value="nodeUrl" v-model="nodeUrl"
                   @focus="$event.target.select()">
              </div>
            </div>
            <h3>Account</h3>
            <p>
              Account addresses are used to sign the transaction.<br />
              The account must have Ether to deploy the contract and set the value.
            </p>
            <div class="form-group">
              <label for="address" class="col-md-4 control-label text-right"></label>
              <div class="col-md-8">
                <button type="button"
                  class="btn btn-primary"
                  @click.prevent="createAccount">Create an account</button>
              </div>
            </div>
            <div class="form-group">
              <label for="privateKey" v-show="!showPass" class="col-md-4 control-label text-right">Private Key</label>
              <label for="privateKeyShow" v-show="showPass" class="col-md-4 control-label text-right">Private Key</label>
              <div class="col-md-7">
                <input
                  type="password"
                  id="privateKey"
                  class="form-control"
                  value="privateKey" v-model="privateKey"
                  v-show="!showPass"
                   @focus="$event.target.select()">
                 <input
                   type="text"
                   id="privateKeyShow"
                   class="form-control"
                   value="privateKey" v-model="privateKey"
                   v-show="showPass"
                    @focus="$event.target.select()">
              </div>
              <div class="col-md-1 text-right">
                 <button type="button"
                   class="btn btn-default"
                   @click.prevent="showPass = !showPass">
                   <span class="glyphicon glyphicon-eye-open" v-show="!showPass" aria-hidden=true></span>
                   <span class="glyphicon glyphicon-eye-close" v-show="showPass" aria-hidden=true></span>
                 </button>
              </div>
            </div>
            <div class="form-group">
              <label for="address" class="col-md-4 control-label text-right">Public Key</label>
              <div class="col-md-7">
                <input
                  type="text"
                  id="address"
                  class="form-control"
                  value="address" v-model="address"
                  ref="addressField"
                  @focus="$event.target.select()" @paste="getBalance(address)">
              </div>
            </div>

            <div class="form-group">
              <div class="col-md-offset-4 col-md-8">
                <div class="alert alert-danger" role="alert">Be careful to only send test Private Key</div>
              </div>
            </div>
            <div class="form-group">
              <label for="privateKey" class="col-md-4 text-right">Balance</label>
              <div class="col-md-8">
                {{ balance }}
              </div>
            </div>
          </form>
          <h3>Access the contact</h3>
          <p>
            Enter an existing contract address or deploy a new one.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="contractAddress" class="col-md-3 control-label text-right">Existing contract address</label>
              <div class="col-md-6">
                <input
                  type="text"
                  id="contractAddress"
                  class="form-control"
                  value="contractAddress" v-model="contractAddress"
                   @focus="$event.target.select()"/>
              </div>
              <div class="col-md-1 text-center">
                Or
              </div>
              <div class="col-md-2">
                <button type="button" v-if="deploying == false"
                  class="btn btn-success"
                  @click.prevent="deployContract">Deploy a contact</button>
                  <img  v-if="deploying == true" width="32" src="@/assets/loading.gif" />
              </div>
            </div>
          </form>
          <div class="row" v-if="deployTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert"><strong>Txhash:</strong> {{ deployTransactionHash }}</div>
            </div>
          </div>
          <h3>New value in the storage</h3>
          <p>
            Interact with the smart contract to set a new storage value.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="storageValue" class="col-md-3 control-label text-right">New storage value</label>
              <div class="col-md-2">
                <input
                  type="text"
                  id="storageValue"
                  class="form-control"
                  value="storageValue" v-model="storageValue" />
              </div>
              <div class="col-md-2">
                <button type="button" v-if="executing == false"
                  class="btn btn-success"
                  @click.prevent="setValue">Set Value</button>
                  <img  v-if="executing == true" width="32" src="@/assets/loading.gif" />
              </div>
            </div>
          </form>
          <div class="row" v-if="execTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert"><strong>Txhash:</strong> {{ execTransactionHash }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

// var solc = require('solc')
import Web3  from 'web3'
import Tx from 'ethereumjs-tx'
import Units from 'ethereumjs-units'

let contractAbi = '[{"constant": false,"inputs": [{"name": "_x","type": "string"}],"name": "set","outputs": [],"payable": false,"stateMutability": "nonpayable","type": "function"},{"constant": true,"inputs": [],"name": "get","outputs": [{"name": "","type": "string"}],"payable": false,"stateMutability": "view","type": "function"},{"anonymous": false,"inputs": [{"indexed": false,"name": "storedData","type": "string"}],"name": "Notify","type": "event"}]'
let contractBytecode = `0x6060604052341561000f57600080fd5b61037f8061001e6000396000f30060606040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680634ed3885e146100515780636d4ce63c146100ae575b600080fd5b341561005c57600080fd5b6100ac600480803590602001908201803590602001908080601f0160208091040260200160405190810160405280939291908181526020018383808284378201915050505050509190505061013c565b005b34156100b957600080fd5b6100c16101f2565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156101015780820151818401526020810190506100e6565b50505050905090810190601f16801561012e5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b806000908051906020019061015292919061029a565b507fc88633c9c082cd7ecb86718d986cac02aafc2ea7a3c91c69226d84f49bec62d1816040518080602001828103825283818151815260200191508051906020019080838360005b838110156101b557808201518184015260208101905061019a565b50505050905090810190601f1680156101e25780820380516001836020036101000a031916815260200191505b509250505060405180910390a150565b6101fa61031a565b60008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156102905780601f1061026557610100808354040283529160200191610290565b820191906000526020600020905b81548152906001019060200180831161027357829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106102db57805160ff1916838001178555610309565b82800160010185558215610309579182015b828111156103085782518255916020019190600101906102ed565b5b509050610316919061032e565b5090565b602060405190810160405280600081525090565b61035091905b8082111561034c576000816000905550600101610334565b5090565b905600a165627a7a723058207f12c8232715f653dabdfaca467ed159d5affb992bef825fa43a7641a0d50a3c0029`

let contractSource = `pragma solidity ^0.4.0;

contract SimpleStorage {
    string storedData;

    event Notify(
       string storedData
    );

    function set(string _x) public {
        storedData = _x;
        Notify(_x);
    }

    function get() public constant returns (string) {
        return storedData;
    }
}`

export default {
  name: 'VueSimpleStorage',

  data () {
    return {
      title: 'VueSimpleStorage with Events',
      msg: 'Simple Storage Ethereum smart contract interface with Vuejs and events',
      contractSource: contractSource,
      nodeUrl: '',
      contractAddress: '',
      address: '',
      privateKey: '',
      balance: '',
      storageValue: '',
      retreivedValue: '',
      deployTransactionHash: '',
      execTransactionHash: '',
      deploying: false,
      executing: false,
      showPass: false
    }
  },
  watch: {
    address: function(val, oldVal) {
      // Get balance
      this.getBalance(val)
    },
    contractAddress: function(val, oldVal) {
      this.subscribeEvent()
    },
    privateKey: function(val, oldVal) {
      // Force 0x addresses
      var regex1 = /(0x)/;
      if (regex1.test(val) == false) {
        val = '0x' + val
      }
      // Extract account
      let web3 = new Web3()
      let account = web3.eth.accounts.privateKeyToAccount(val);
      this.address = account.address
    }
  },
  created() {
    this.subscribeEvent()

    // Check balance
    if (this.address != '') {
      this.getBalance(this.address)
    }
  },
  methods: {
    // Dial node
    dialNode() {
      // New web3
      let web3 = new Web3()
      // Get provider
      let url = this.nodeUrl
      web3.setProvider(this.nodeUrl)

      return web3
    },
    // Get balance
    getBalance(address) {
      // New web3
      let web3 = new Web3()
      if (web3.utils.isAddress(address)) {
        // Dial node
        web3 = this.dialNode()

        // Get balance
        let self = this
        web3.eth.getBalance(address)
        .then(function(balance){
          console.log('Get balance: ' + balance)
          balance =  Units.convert(balance, 'wei', 'eth')
          self.balance = balance + ' Ether'
        })
      } else {
        console.log('Address not valid')
        return
      }
    },
    createAccount() {
      console.log('Create account')

      // Dial node
      let web3 = this.dialNode()

      // Create account
      let account = web3.eth.accounts.create();
      console.log(account)

      // Set account info
      this.address = account.address
      this.privateKey = account.privateKey

      // Get balance
      this.getBalance(account.address)
    },
    // Deploy contract
    deployContract() {
      console.log('Deploy contract on node: ' + this.nodeUrl + ', for address: ' + this.address)

      // Dial node
      let web3 = this.dialNode()

      // Get keys
      var privateKey = new Buffer(this.privateKey, 'hex')

      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi);
      var contractDeploy = contract.deploy({
          data: contractBytecode
          //arguments: args
      })
      console.log('Contract: ', contract);
      // Get payload
      var payload = contractDeploy.encodeABI();

      // Save current this
      let self = this
      self.deploying = true
      self.contractAddress = ''

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log('Get GasPrice error: ', gasPriceError)
          self.deploying = false
          return
        } else{
          var gasPrice = result;
          console.log('Get GasPrice success: ', gasPrice)
          var gasPriceHex = web3.utils.numberToHex(gasPrice)

          // Get nonce
          web3.eth.getTransactionCount(self.address, function(nonceError, nonce) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError)
              self.deploying = false
              return
            }
            else {
              // Get the current nonce
              const nonceHex = web3.utils.numberToHex(nonce)
              console.log("Nonce hex : ", nonce)

              // Set the gas limit
              const gasLimitHex = web3.utils.numberToHex(300000)

              // Create transaction
              const rawTx = {
                nonce: nonceHex,
                gasPrice: gasPriceHex,
                gasLimit: gasLimitHex,
                value: '0x00',
                from: self.address,
                data: payload
              }
              console.log("Raw Transaction: ",rawTx)

              // Sign and serialize the transaction
              console.log("Sign raw transaction with privatekey: ", privateKey)
              const tx = new Tx(rawTx)
              tx.sign(privateKey)
              const serializedTx = tx.serialize();
              console.log("Raw transaction ready to be sent: ", "0x" + serializedTx.toString('hex'))

              // Send signed transaction
              web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'))
                .on('error', function(error){
                  console.log('sendRawTransaction error: ', error)
                  self.deploying = false
                })
                .on('transactionHash', function(transactionHash){
                  console.log('sendRawTransaction success: ', transactionHash)
                  self.deployTransactionHash = transactionHash
                })
                .on('receipt', function(receipt){
                  console.log('Receipt: ', receipt.contractAddress)
                  self.contractAddress = receipt.contractAddress
                  self.deploying = false
                })
                // .on('confirmation', function(confirmationNumber, receipt){
                //   console.log(confirmationNumber)
                //   console.log(receipt)
                // })
                // .then(function(newContractInstance){
                //   console.log(newContractInstance) // instance with the new contract address
                // })
            }
          })
        }
      })
    },
    // Set storage value
    setValue() {
      console.log('Set value:' + this.storageValue + ' to contract: ' + this.contractAddress + ' on node: ' + this.nodeUrl)

      // Dial node
      let web3 = this.dialNode()

      // Get keys
      var privateKey = new Buffer(this.privateKey, 'hex')

      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress)
      console.log('Contract: ', contract);

      // Save current this
      let self = this
      self.executing = true

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log('Get GasPrice error: ', gasPriceError)
          self.executing = false
          return
        } else{
          var gasPrice = result;
          console.log('Get GasPrice success: ', gasPrice)
          var gasPriceHex = web3.utils.numberToHex(gasPrice)

          // Get nonce
          web3.eth.getTransactionCount(self.address, function(nonceError, nonce) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError)
              self.executing = false
              return
            }
            else {
              // Get the current nonce
              const nonceHex = web3.utils.numberToHex(nonce)
              console.log("Nonce hex : ", nonce)

              // Execute the smart contract method
              var newValue = self.storageValue
              var contractCallData = contract.methods.set(newValue)
              console.log('contractCallData : ', contractCallData)

              // Get payload
              var payload = contractCallData.encodeABI()

              // Set the gas limit
              const gasLimitHex = web3.utils.numberToHex(3000000)

              // Create transaction
              const rawTx = {
                nonce: nonceHex,
                gasPrice: gasPriceHex,
                gasLimit: gasLimitHex,
                value: '0x00',
                from: self.address,
                to: self.contractAddress,
                data: payload
              }
              console.log("Raw Transaction: ",rawTx)

              // Sign and serialize the transaction
              console.log("Sign raw transaction with privatekey: ", privateKey)
              const tx = new Tx(rawTx)
              tx.sign(privateKey)
              const serializedTx = tx.serialize();
              console.log("Raw transaction ready to be sent: ", "0x" + serializedTx.toString('hex'))

              // Send signed transaction
              web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'))
                .on('error', function(error){
                  console.log('sendRawTransaction error: ', error)
                  self.executing = false
                })
                .on('transactionHash', function(transactionHash){
                  console.log('sendRawTransaction success: ', transactionHash)
                  self.execTransactionHash = transactionHash
                })
                .on('receipt', function(receipt){
                  console.log("receipt: ", receipt)
                  self.executing = false
                })
                .on('confirmation', function(confirmationNumber, receipt){
                  console.log('Confirmation: ', confirmationNumber, ', Receipt: ', receipt)
                })
                // .then(function(newContractInstance){
                //   console.log(newContractInstance) // instance with the new contract address
                // })
            }
          })
        }
      })
    },
    subscribeEvent() {
      if (this.contractAddress != '') {
        console.log('Subscribe event:', this.contractAddress);
        // Dial node
        let web3 = this.dialNode()

        var abi = JSON.parse(contractAbi)
        var contract = new web3.eth.Contract(abi, this.contractAddress)

        // Catch event
        let self = this
        contract.events.Notify( {},
          { fromBlock: 'latest' },
          function(error, event){
            if (event != null && typeof event.returnValues[0] != 'undefined') {
              let value = event.returnValues[0]
              console.log("Contract value:", value);
              self.retreivedValue = value
            }
          })
      } else {
        console.log('No contract address to subscribe')
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

h1, h2 {
  font-weight: normal;
}

</style>
