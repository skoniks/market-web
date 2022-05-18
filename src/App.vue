<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue';

type Market = {
  name: string;
  price: number;
};

type Steamp = {
  price: number;
  appid: string;
  count: number;
  bprice: number;
  bcount: number;
};

type Item = {
  name: string;
  market_price: number;
  steam_price: number;
  steam_count: number;
  steam_bprice: number;
  steam_bcount: number;
  profit_price: number;
  profit_bprice: number;
};

const steampKey = ref(localStorage.getItem('steamp') || '');
watch(steampKey, (key) => localStorage.setItem('steamp', key));

const data = reactive({
  marketItems: [] as Market[],
  steampItems: {} as { [name: string]: Steamp },
});

const filters = reactive({
  market_price: { from: 0, to: 0, max: 100 },
  steam_price: { from: 0, to: 0, max: 0 },
  steam_count: { from: 0, to: 0, max: 0 },
  steam_bprice: { from: 0, to: 0, max: 0 },
  steam_bcount: { from: 0, to: 0, max: 0 },
  profit_price: { from: 0, to: 0, max: 0 },
  profit_bprice: { from: 0, to: 0, max: 0 },
});

const items = computed(() =>
  data.marketItems.reduce((items, market) => {
    const steamp = data.steampItems[market.name];
    if (steamp) {
      items.push({
        name: market.name,
        market_price: market.price,
        steam_price: steamp.price,
        steam_count: steamp.count,
        steam_bprice: steamp.bprice,
        steam_bcount: steamp.bcount,
        profit_price: steamp.price / market.price,
        profit_bprice: steamp.bprice / market.price,
      });
    }
    return items;
  }, [] as Item[]),
);

const sort = ref('');
const sortBy = ref(0);
const sorted = computed(() =>
  items.value.slice(0).sort((a, b) => {
    if (!sort.value || !sortBy.value) return 0;
    const A = a[sort.value as keyof Item];
    const B = b[sort.value as keyof Item];
    return (A > B ? 1 : -1) * sortBy.value;
  }),
);

// const filtered = computed(() => sorted.value.slice(0, 30));

const changeSort = (value: string) => {
  if (sort.value !== value) {
    sort.value = value;
    sortBy.value = -1;
  } else if (sortBy.value == -1) {
    sortBy.value = 1;
  } else {
    sort.value = '';
    sortBy.value = 0;
  }
};

const marketUpdate = async () => {
  try {
    const response = await fetch('http://market.sknx.ru:8800/market').then(
      (response) => response.json(),
    );
    if (response.success) {
      data.marketItems.splice(0);
      data.marketItems.push(
        ...response.items.map((i: any) => ({
          name: i.market_hash_name,
          price: Number(i.price),
        })),
      );
    } else throw new Error();
  } catch (error) {
    alert('Ошибка получения предметов с маркета');
  }
};

const steampUpdate = async () => {
  try {
    const response = await fetch(
      'http://market.sknx.ru:8800/steamp/' + steampKey.value,
    ).then((response) => response.json());
    if (response.success) {
      data.steampItems = response.items;
    } else {
      alert('Ошибка получения предметов со Steamp\n' + response.message);
    }
  } catch (error) {
    alert('Ошибка получения предметов со Steamp');
  }
};
</script>

<template>
  <header>
    <input type="text" v-model="steampKey" placeholder="Steamp API" />
    <p>
      <span>Предметов на маркете: {{ data.marketItems.length }}</span>
      <a href="#" @click="marketUpdate()">Обновить Market</a>
    </p>
    <p>
      <span
        >Предметов на Steamp: {{ Object.keys(data.steampItems).length }}</span
      >
      <a href="#" @click="steampUpdate()">Обновить Steamp</a>
    </p>

    <div class="filters">
      <div>
        <span>Цена на маркете</span>
        <div class="ranges">
          <input
            type="range"
            :min="0"
            :max="filters.market_price.max"
            v-model="filters.market_price.from"
          />
          <input
            type="range"
            :min="0"
            :max="filters.market_price.max"
            v-model="filters.market_price.to"
          />
        </div>
      </div>
    </div>
  </header>
  <main>
    <table>
      <tr>
        <th>Название</th>
        <th @click="changeSort('market_price')">Цена на маркете</th>
        <th @click="changeSort('steam_price')">Цена в стиме</th>
        <th @click="changeSort('steam_bprice')">Цена автозакупа</th>
        <th @click="changeSort('steam_count')">Запросов на продажу</th>
        <th @click="changeSort('steam_bcount')">Запросов на покупку</th>
        <th @click="changeSort('profit_price')">Профит по цене</th>
        <th @click="changeSort('profit_bprice')">Профит автозакуп</th>
      </tr>
      <tr v-for="item in sorted">
        <td>{{ item.name }}</td>
        <td>{{ item.market_price }}</td>
        <td>{{ item.steam_price }}</td>
        <td>{{ item.steam_bprice }}</td>
        <td>{{ item.steam_count }}</td>
        <td>{{ item.steam_bcount }}</td>
        <td>{{ item.profit_price.toFixed(2) }}</td>
        <td>{{ item.profit_bprice.toFixed(2) }}</td>
      </tr>
    </table>
  </main>
</template>

<style lang="scss">
#app {
  max-width: 1280px;
  margin: 0 auto;
  padding: 2rem;
  font-weight: normal;
}
header {
  .filters {
    .ranges {
      //
    }
  }
}
</style>
