<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <div class="web">
      <div class="web_text">
        Версия web3.js: {{ version }}</br>

      </div>
      <div class="web__list">
        <h4>Создать аккаунт</h4>
        <div class="web__list_item">
          <div class="web__list_item-left">
            <div class="">пароль</div>
            <input type="text" v-model="passAccounts">
          </div>
          <button @click="createAccounts()">Создать</button>
        </div>
      </div>
      <div class="web__list">
        <h4>Список аккаунтов</h4>
        <div class="web__list_item" v-for="(item, index) in list">
          <div class="web__list_item-left">{{ index + 1 }}. {{ item.name }}</div>
          <div class="web__list_item-right">Баланс: {{ item.balance }} ether</div>
        </div>
      </div>
      <div class="web__list">
        <h4>Перевод ether</h4>
        <div class="web__list_input">
          <div>from:</div>
          <input type="text" v-model="fromTrans">
        </div>
        <div class="web__list_input">
          <div>to:</div>
          <input type="text" v-model="toTrans">
        </div>
        <div class="web__list_number">
          <div>value:</div>
          <input type="number" v-model="valueTrans">
        </div>
        <div class="web__list_number">
          <div>пароль:</div>
          <input type="text" v-model="valuePass">
        </div>
        <button class="button" @click="sendTransaction()" >Перевести</button>
      </div>
    </div>
  </div>
</template>

<script>

  //  import Personal from 'web3-eth-personal';
  //  const personal = new Personal(Personal.givenProvider || 'ws://localhost:8546');

  import Web3 from 'web3'
  import lightwallet from 'eth-lightwallet'

  const web3 = new Web3(Web3.givenProvider || 'ws://localhost:8546');

  import fs from 'fs'


  export default {
    name: 'HelloWorld',
    data () {
      return {
        msg: 'Welcome to Your Smart Contract App',
        account: '',
        version: [],
        list: [],
        toTrans:'',
        fromTrans: '',
        valueTrans: 0,
        passAccounts: '',
        valuePass: ''
      }
    },
    computed: {

    },
    methods: {
      createAccounts(){
        let self = this
        web3.eth.personal.newAccount(this.passAccounts)
          .then(
            function (response) {
              console.log(response);
              self.passAccounts = '';
              self.start()
            }
          );
      },
      sendTransaction() {
        let self = this;
        web3.eth.personal.unlockAccount(self.fromTrans, self.valuePass, 600)
          .then(
            function (response) {

              const ether = web3.utils.toWei(self.valueTrans , 'ether')

              web3.eth.sendTransaction(
                {
                  from: self.fromTrans,
                  to: self.toTrans,
                  value: ether
                }
              ).then(
                function (response) {
                  self.start()
                }
              )
            }
          );

      },
      eth() {
        const keyStore = lightwallet.keystore;
        const password = prompt('Enter password for encryption', 'password');

        keyStore.createVault({
          password: password,
          // seedPhrase: seedPhrase, // Optionally provide a 12-word seed phrase
          // salt: fixture.salt,     // Optionally provide a salt.
          // A unique salt will be generated otherwise.
          // hdPathString: hdPath    // Optional custom HD Path String
        }, function (err, ks) {
          console.log(ks)
          ks.keyFromPassword(password, function (err, pwDerivedKey) {
            if (err) throw err;

            ks.generateNewAddress(pwDerivedKey, 5);
            const addr = ks.getAddresses();
            console.log(addr)
            ks.passwordProvider = function (callback) {
              const pw = prompt("Please enter password", "Password");
              callback(null, pw);
            };
          });
        });
      },
      start(){
        let self = this;
        self.list = [];
        web3.eth.getAccounts()
          .then(
            function (response) {
              response.forEach(function (item, i, arr) {
                const balance = web3.eth.getBalance(item)
                  .then(
                    function (response) {
                      const wei = web3.utils.fromWei(response , 'ether')
                      self.list.push({
                        name: item,
                        balance: wei
                      });
                    }
                  );
              });
            }
          );

        this.version = web3.version
      },
      Balance() {
        web3.eth.getBalance(this.account,
          function (err, res) {
            console.log(res)
          }
        )
      },
    },
    mounted() {
      let self = this
      self.start()
//      this.eth()
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }
  .button{
    margin: 20px;
  }
  .web__list{
    text-align: left;
    padding: 40px;

  }
  .web__list_number{
    width: 500px;
    margin-top: 20px;
  }
  .web__list_input{
    display: inline-block;
    width: 500px;
  }
  input{
    width: 80%;
  }
  h4{
    font-size: 26px;
    margin: 10px;
  }
  .web__list_item{
    padding: 3px 0px;
    display: flex;
  }
  .web__list_item-right{
    padding-left: 20px;
  }
  .web_text {
    padding-top: 10px;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
