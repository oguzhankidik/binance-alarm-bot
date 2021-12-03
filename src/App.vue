<template>
  <div id="app">
   <div class="row filter">
     <label class="col-1"> Symbol: </label>
     <input type="text" v-model="symbol" class="col">
     <label class="col-1"> Limit: </label>
     <input type="text" v-model="limit" class="col">
     <button type="button" class="btn btn-primary ml-2" @click="updateTrades">Start</button>
     <button type="button" class="btn btn-danger ml-2" @click="stopBot">Stop</button>

   </div>
    <div>
      <div v-for="(items,index) in tradeList" :key="index">
        <span class="buyer" :class="{seller : items.type}">price:  {{ items.price }} </span>
        <span :class = "{active : checkLimit(items)}">qty:   {{ items.qty }} </span>
      </div>
    </div>

  </div>
</template>

<script>
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";


export default {
  name: 'App',
  data: () => ({
    url: 'https://api.binance.com/',
    answer: '',
    tradeList : [],
    symbol : 'BNBUSDT',
    limit : null,
    interval : null,
    firebaseConfig : {},
    app : null,
    analytics : null
  }),

  computed:{

  },

   mounted() {
     this.firebaseConfig = {
       apiKey: "AIzaSyAI-yup4qYvSMFtt0nlQfU0XN7Kq7lZPCc",
       authDomain: "binance-alert-bot.firebaseapp.com",
       projectId: "binance-alert-bot",
       storageBucket: "binance-alert-bot.appspot.com",
       messagingSenderId: "35516541304",
       appId: "1:35516541304:web:453cc975531c714bbcc08d",
       measurementId: "G-J15MSXZTHF"
     };
     this.app = initializeApp(this.firebaseConfig);
     this.analytics = getAnalytics(this.app);
  },

  methods: {
    updateTrades(){
     this.interval = window.setInterval(this.getBinance,1000)
    },
    stopBot(){
      window.clearInterval(this.interval)
    },

    async getBinance() {
      let query = 'api/v3/trades'
      let params = {
        symbol: this.symbol,
        limit: 50
      }

      this.answer = await this.$axios.get(this.url + query, { params })
      this.prepareData(this.answer.data)
    },

    prepareData(data) {
      this.tradeList = []
      data.forEach(item => {
        let b = {
          price : item.price,
          qty : item.qty,
          time : new Date(data.time).toLocaleString(),
          id : item.id,
          type : item.isBuyerMaker
        }
        this.tradeList.push(b)
      })
      this.tradeList.reverse()
    },

    checkLimit(item){
      let value = this.round(item.qty,1)
      return value > this.limit
    },
    round(value, precision) {
      let multiplier = Math.pow(10, precision || 0);
      return Math.round(value * multiplier) / multiplier;
    }
  }
}
</script>

<style lang="scss">
#app{
  padding: 20px;
  margin: auto;
  width: 50%;
}
span{

  padding: 10px;
  margin-right: 5px;
  text-align: center;
  line-height: 3;
}
.active{
  background-color: red;
}
.seller{
  color: red !important;
}
.buyer{
  color: green;
}
.filter{
  label{
    margin-top: 10px;
  }
}
</style>
