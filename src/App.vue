<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div class="fixed w-100 h-100 opacity-80 bg-purple-800 inset-0 z-50 flex items-center justify-center" v-if="!appLoaded">
      <svg class="animate-spin -ml-1 mr-3 h-12 w-12 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
    </div>

    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700"> Тикер </label>
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="ticker"
                @keydown.enter="add"
                @input="searchCoin"
                type="text"
                name="wallet"
                id="wallet"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                placeholder="Например DOGE"
              />
            </div>
            <div class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap" v-if="findCoins.length > 0">
              <span
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
                v-for="(coin, key) of findCoins"
                :key="key"
                @click="
                  {
                    ticker = coin;
                    add();
                  }
                "
              >
                {{ coin }}
              </span>
            </div>
            <div v-if="alreadyTicker" class="text-sm text-red-600">Такой тикер уже добавлен</div>
          </div>
        </div>
        <button
          @click="add"
          type="button"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          <!-- Heroicon name: solid/mail -->
          <svg class="-ml-0.5 mr-2 h-6 w-6" xmlns="http://www.w3.org/2000/svg" width="30" height="30" viewBox="0 0 24 24" fill="#ffffff">
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>
      <template v-if="this.tickers.length">
        <div>
          <button
            :disabled="page <= 1"
            @click="page--"
            class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-l disabled:opacity-50"
          >
            Назад
          </button>
          <button
            :disabled="btnDisabled"
            @click="page++"
            class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-r disabled:opacity-50"
          >
            Вперед
          </button>
          <div class="bg-gray-300 mt-4 h-10 rounded-lg flex gap-4 items-center justify-between px-4 w-1/5 font-bold">
            Фильтр: <input v-model="filter" class="bg-gray-200 h-full border-2 border-solid px-4 border-gray-300" />
          </div>
        </div>
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="t of paginatedTickers"
            :key="t.name"
            @click="select(t)"
            :class="{
              'border-dashed border-2': sel === t,
            }"
            class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer flex flex-col "
          >
            <div class="px-4 py-5 sm:p-6 text-center flex-grow">
              <dt class="text-sm font-medium text-gray-500 truncate">{{ t.name }} - USD</dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">{{ t.price }} USD</dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              @click.stop="handleDelete(t)"
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
            >
              <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="#718096" aria-hidden="true">
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path></svg
              >Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>
      <section v-if="sel" class="relative">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">{{ sel.name }} - {{ sel.price }} USD</h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64">
          <div v-for="(bar, idx) in normalizedGraph" :key="idx" :style="{ height: `${bar}%` }" class="bg-purple-800 border w-8 custom-graph"></div>
        </div>
        <button type="button" @click="sel = null" class="absolute top-0 right-0">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs"
            version="1.1"
            width="30"
            height="30"
            x="0"
            y="0"
            viewBox="0 0 511.76 511.76"
            style="enable-background:new 0 0 512 512"
            xml:space="preserve"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096"
                data-original="#000000"
              ></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>
<script>
export default {
  name: 'App',

  data() {
    return {
      filter: '',
      ticker: '',

      tickers: [],
      sel: null,

      alreadyTicker: false,
      graph: [],
      allCoins: [],
      findCoins: [],
      appLoaded: true,
      page: 1,
      tickersOnPage: 6,
    };
  },

  created() {
    const windowData = Object.fromEntries(new URL(window.location).searchParams.entries());

    if (windowData.filter) {
      this.filter = windowData.filter;
    }

    if (windowData.page) {
      this.page = windowData.page;
    }

    const tickersData = localStorage.getItem('cryptonomicon-list');
    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach((ticker) => {
        this.subscribeToUpdates(ticker.name);
      });
    }
  },

  computed: {
    startIndex() {
      return (this.page - 1) * this.tickersOnPage;
    },
    endIndex() {
      return this.page * this.tickersOnPage;
    },

    filteredTickers() {
      return this.tickers.filter((ticker) => ticker.name.includes(this.filter.toUpperCase()));
    },

    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },

    btnDisabled() {
      return this.tickers.length <= this.endIndex;
    },
    normalizedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);
      return this.graph.map((price) => (minValue === maxValue ? 100 : 5 + ((price - minValue) * 95) / (maxValue - minValue)));
    },
  },

  methods: {
    subscribeToUpdates(tickerName) {
      setInterval(async () => {
        const f = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=046bb019f05540522ba573c299b93ce437555d72d2fabf2b52942a7af2f31132`
        );
        const data = await f.json();
        this.tickers.find((t) => t.name === tickerName).price = data.USD > 1 ? data.USD.toFixed(2) : data.USD?.toPrecision(2);
        if (this.sel?.name === tickerName) this.graph.push(data.USD);
        if (this.graph.length > 80) this.graph.shift();
      }, 3000);
    },

    add() {
      if (this.ticker === '') return false;
      const currentTicker = {
        name: this.ticker.toUpperCase(),
        price: '',
      };
      this.alreadyTicker = false;
      this.tickers.some((elem) => elem.name == currentTicker.name) ? (this.alreadyTicker = true) : this.tickers.push(currentTicker);
      this.subscribeToUpdates(currentTicker.name);
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
      this.ticker = '';
      this.filter = '';
    },

    select(ticker) {
      (this.sel = ticker), (this.graph = []);
    },

    handleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter((t) => t !== tickerToRemove);
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
    },

    searchCoin() {
      const coincidence = this.allCoins;
      const inpText = this.ticker.toUpperCase();
      this.findCoins = [];
      this.alreadyTicker = false;
      for (let k in coincidence) {
        if (k.includes(inpText)) {
          if (this.findCoins.length < 4) this.findCoins.push(k);
          if (k === inpText) {
            this.findCoins.shift();
            this.findCoins.unshift(k);
          }
        }
      }
    },
  },

  mounted() {
    window.addEventListener('load', async () => {
      this.appLoaded = true;
      const fetchCoins = await fetch('https://min-api.cryptocompare.com/data/all/coinlist?summary=true');
      const data = await fetchCoins.json();
      this.allCoins = data.Data;
    });
  },

  watch: {
    filter() {
      this.page = 1;
      window.history.pushState(null, document.title, `${window.location.pathname}?filter=${this.filter}&page=${this.page}`);
    },
    page() {
      window.history.pushState(null, document.title, `${window.location.pathname}?filter=${this.filter}&page=${this.page}`);
    },
  },
};
</script>

<style src="./app.css"></style>
<style>
.custom-graph {
  transition: 0.5s all;
}
</style>
