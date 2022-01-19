<template>
  <div class="hello">
    <div>此页面用于测试前端通过web3.js与metamask相连从而调用智能合约</div>
    <div>请使用以太坊测试网rinkeby参与测试</div>
    <div>点击add，成功发送交易后将使存储数据加1，点击reduce，成功发送交易后将使数据减1</div>
    <div>
    您的账号地址为：  {{ accounts }}
    </div>

    <div>
   当前存储数据为：   {{ number }}
    </div>

    <button v-on:click="add()">add</button>

    <button v-on:click="reduce()">reduce</button>
  </div>
</template>

<script>
const Web3 = require("web3");
import ABIJson from "./StorageABI.json";

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },

  data() {
    return {
      title: "jiba",
      accounts: "",
      contract: null,
      number: 0,
    };
  },

  onload() {},
  async mounted() {
    let that = this;

    let P = new Promise((resolve, reject) => {
      const timer = setInterval(async () => {
        var web3Provider;
        if (window.ethereum) {
          web3Provider = window.ethereum;
          try {
            const accounts = await window.ethereum.request({
              method: "eth_requestAccounts",
            });
            var web3js = new Web3(web3Provider);
            web3js.eth.defaultAccount = accounts[0];
            clearInterval(timer);
            console.dir(accounts);
            resolve({ web3: web3js, loggedIn: true });
          } catch (error) {
            console.error("User denied account access");
          }
        } else if (window.web3) {
          web3Provider = window.web3.currentProvider;
          var web3js2 = new Web3(web3Provider);
          web3js2.eth.getAccounts(function (error, result) {
            if (error) {
              console.log(error);
              reject("User denied account access", false);
            }
            console.dir(result);
            web3js2.eth.defaultAccount = result[0];
            clearInterval(timer);
            resolve({ web3: web3js2, loggedIn: true });
          });
        }
      }, 100);
    });
    P.then(async (data) => {
      that.accounts = data.web3.eth.defaultAccount;
      that.contract = new data.web3.eth.Contract(
        ABIJson,
        "0x191a1c48431160cf4f0e86cb7ac868b81555f493"
      );
      that.number = Number(await that.contract.methods.retrieve().call());
      console.dir(that.number);
    }).catch((e) => {
      that.accounts = e;
    });
  },
  methods: {
    async add() {
      let that = this;
      if (that.number >= 0) {
        await that.contract.methods
          .store(that.number + 1)
          .send({ from: that.accounts });
        that.number = Number(await that.contract.methods.retrieve().call());
      }
    },
    async reduce() {
      let that = this;
      if (that.number > 0) {
        await that.contract.methods
          .store(that.number - 1)
          .send({ from: that.accounts });
        that.number = Number(await that.contract.methods.retrieve().call());
      }
    },
  },
};
</script>

<style>
button{
  display: block;
  width: 100px;
  height: 50px;
  margin: 10px auto;
}

div{
  margin: 10px auto;
}


</style>
