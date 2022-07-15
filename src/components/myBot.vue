<template>
  <div id="myBot" v-if="notDeleted">
    <div v-show="loading" class="overlay"></div>
    <div class="filter">
      <div>
        <label> Symbol: </label>
        <input type="text" v-model="symbol" />
      </div>
      <div>
        <label> Limit: </label>
        <input type="text" v-model="limit" />
      </div>
      <div>
        <label> Total Limit: </label>
        <input type="text" v-model="maxLimit" />
      </div>
      <div>
        <button
          type="button"
          :disabled="clicked"
          class="btn btn-primary ml-2"
          @click="updateTrades"
        >
          Start
        </button>
        <button type="button" class="btn btn-danger ml-2" @click="stopBot">
          Stop
        </button>
      </div>
    </div>
    <div class="binance-table">
      <div v-if="tradeList.length" class="row table-headers">
        <span class="col">Price</span>
        <span class="col">Quantity</span>
        <span class="col">Total Buy: {{ count }}</span>
      </div>

      <div v-for="(items, index) in tradeList" :key="index" class="row">
        <span class="buyer col-4" :class="{ seller: items.type }"
          >{{ items.price }}
        </span>
        <span class="col-4" :class="{ 'limit-exceed': checkLimit(items) }"
          >{{ items.qty }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import Push from "push.js";

export default {
  name: "myBot",
  data: () => ({
    url: "https://api.binance.com/",
    answer: "",
    tradeList: [],
    symbol: "BNBUSDT",
    limit: null,
    interval: null,
    loading: false,
    clicked: false,
    addButton: true,
    notDeleted: true,
    lastTrades: [],
    count: 0,
    ids: [],
    popOutCount: 0,
    maxLimit: null,
    staticLength: 0,
  }),

  watch: {
    popOutCount: {
      handler(val) {
        if (val > 5) {
          this.popOutFromTrades();
        }
        if (val === 5) {
          this.staticLength = this.lastTrades.length;
        }
      },
    },

    count: {
      handler(val) {
        if (val > this.maxLimit) {
          Push.create("WAKE UP!! MAX LIMIT EXCEEDED!", {
            body: `${this.symbol}: Total Buy: ${val} `,
            requireInteraction: true,
            onClick: function () {
              window.focus();
              this.close();
            },
          });
        }
      },
    },
  },
  props: {
    tabCount: {
      type: Number,
      default: 1,
    },
  },

  methods: {
    updateTrades() {
      this.loading = true;
      this.clicked = true;
      this.interval = window.setInterval(this.getBinance, 2000);
    },
    stopBot() {
      window.clearInterval(this.interval);
      this.clear();
      this.clicked = false;
      Push.clear();
    },

    async getBinance() {
      let query = "api/v3/trades";
      let params = {
        symbol: this.symbol.toUpperCase(),
        limit: 50,
      };
      if (!this.limit) {
        this.limit = 999999;
      }
      if (!this.maxLimit) {
        this.maxLimit = 999999;
      }
      this.answer = await this.$axios.get(this.url + query, { params });
      this.prepareData(this.answer.data);
    },

    prepareData(data) {
      this.tradeList = [];
      data.forEach((item) => {
        let b = {
          price: item.price,
          qty: item.qty,
          time: new Date(data.time).toLocaleString(),
          id: item.id,
          type: item.isBuyerMaker,
        };
        this.tradeList.push(b);
      });
      this.tradeList.reverse();
      this.countBuyers();
      this.loading = false;
    },

    checkLimit(item) {
      if (!item.type) {
        let value = this.round(item.qty, 1);
        if (value > this.limit) {
          Push.create("Wake UP!", {
            body: `Someone bought ${this.symbol} ${item.qty}`,
            requireInteraction: true,
            onClick: function () {
              window.focus();
              this.close();
            },
          });
        }
        return value > this.limit;
      }
      return false;
    },
    round(value, precision) {
      let multiplier = Math.pow(10, precision || 0);
      return Math.round(value * multiplier) / multiplier;
    },

    countBuyers() {
      this.tradeList.forEach((item) => {
        if (!item.type && !this.ids.includes(item.id)) {
          this.lastTrades.push([item.id, item.qty]);
          this.ids.push(item.id);
          this.count += Number(item.qty);
        }
      });
      this.lastTrades.sort((a, b) => {
        return b[0] - a[0];
      });
      this.ids.sort((a, b) => {
        return b - a;
      });
      this.popOutCount += 1;
    },

    popOutFromTrades() {
      let a = this.lastTrades.length - this.staticLength;
      if (a > 0) {
        let b = this.lastTrades.slice(
          this.staticLength,
          this.lastTrades.length
        );
        b.forEach((item) => {
          this.count -= Number(item[1]);
        });
      }
      this.lastTrades = this.lastTrades.slice(0, this.staticLength);
      this.ids = this.ids.slice(0, this.staticLength);
    },

    clear() {
      this.count = 0;
      this.tradeList = [];
      this.lastTrades = [];
      this.ids = [];
      this.popOutCount = 0;
      this.staticLength = 0;
    },
  },
};
</script>

<style lang="scss">
input {
  background-color: #d2d2d2;
}
.overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: 100;
  background-color: black;
  opacity: 0.45;
  backdrop-filter: blur(3);
}
body {
  background-color: #2b2b2b;
  color: white;
}

.seller {
  color: red !important;
}

.buyer {
  color: green;
}
.limit-exceed {
  background: red;
}

.filter {
  display: flex;
  justify-content: center;
  gap: 1rem;
  & label {
    margin-right: 0.25rem;
  }
  & input {
    height: 40px;
  }
}

.binance-table {
  width: 50%;
  display: flex;
  flex-direction: column;
  margin-left: auto;
  margin-right: auto;
}

.table-row {
  height: 30px;
}
.table-headers {
  border-bottom: 1px solid white;
}
</style>
