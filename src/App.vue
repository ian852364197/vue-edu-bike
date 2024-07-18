<script setup>
import { computed, onMounted, ref, watch } from 'vue';
import Paginate from 'vuejs-paginate-next';

//API取得的資料
const bikeList = ref([]);

//分頁用
const page = ref(1);
const pageRangeList = ref([10, 20]);
const pageRange = ref(pageRangeList.value[0]);
const pageCount = computed(() => {
  return parseInt(filterList.value.length / pageRange.value) + 1;
});

//排序用
const rentSort = ref(null);
const totalSort = ref(null);
const rentIcon = computed(() => compIcon(rentSort));
const totalIcon = computed(() => compIcon(totalSort));

//搜尋用
const address = ref('');

//經過搜尋or排序的資料
const filterList = ref([]);
//畫面顯示的資料
const showList = computed(() => {
  let first = (page.value - 1) * pageRange.value;
  let last = page.value * pageRange.value;
  return filterList.value.slice(first, last);
});

//變換排序圖示
function compIcon(sortBy) {
  if (!sortBy.value) {
    return '△';
  }
  return sortBy.value === 'desc' ? '▲' : '▼';
}

//排序方法
function sortByRent() {
  totalSort.value = null;
  if (rentSort.value === 'desc') {
    rentSort.value = 'asc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.available_rent_bikes - b.available_rent_bikes;
    });
  } else {
    rentSort.value = 'desc';
    filterList.value = filterList.value.sort((a, b) => {
      return b.available_rent_bikes - a.available_rent_bikes;
    });
  }
}
function sortByTotal() {
  rentSort.value = null;
  if (totalSort.value === 'desc') {
    totalSort.value = 'asc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.total - b.total;
    });
  } else {
    totalSort.value = 'desc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.total * -1 - b.total * -1;
    });
  }
}

//call api取得資料
onMounted(async () => {
  const test = await fetch(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  test.json().then((value) => {
    bikeList.value = value;
    filterList.value = value;
  });
});

//選擇一頁顯示幾筆
watch(
  () => pageRange.value,
  () => {
    if (page.value !== 1) {
      let now = pageRange.value === 10 ? page.value * 2 : parseInt(page.value / 2);
      page.value = now;
    }
  },
  { immediate: true }
);

//模糊搜尋
watch(
  () => address.value,
  () => {
    page.value = 1;
    rentSort.value = null;
    totalSort.value = null;
    let addText = address.value.trim();
    filterList.value = bikeList.value.filter((station) => station.ar.includes(addText));
  },
  { immediate: true }
);
</script>

<template>
  <div class="container">
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
      <div class="container-fluid">
        <p class="navbar-brand">Bike 練習</p>
        <div class="d-flex">
          <input
            class="form-control me-2"
            type="search"
            placeholder="輸入站點地址"
            aria-label="Search"
            v-model="address"
          />
          <button class="btn btn-outline-success" type="button">Search</button>
        </div>
      </div>
    </nav>
    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col" class="td">站點編號</th>
          <th scope="col" class="td">站點名稱</th>
          <th scope="col" class="td">站點所在區域</th>
          <th scope="col" class="td">站點地址</th>
          <th scope="col" class="td" @click="sortByTotal">總車位數量 {{ totalIcon }}</th>
          <th scope="col" class="td" @click="sortByRent">可租借的腳踏車數量 {{ rentIcon }}</th>
          <th scope="col" class="td">站點緯度</th>
          <th scope="col" class="td">站點經度</th>
          <th scope="col" class="td">可歸還的腳踏車數量</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="station in showList" :key="station.sno">
          <th scope="row">{{ station.sno }}</th>
          <td>{{ station.sna }}</td>
          <td>{{ station.sarea }}</td>
          <td>{{ station.ar }}</td>
          <td>{{ station.total }}</td>
          <td>{{ station.available_rent_bikes }}</td>
          <td>{{ station.latitude }}</td>
          <td>{{ station.longitude }}</td>
          <td>{{ station.available_return_bikes }}</td>
        </tr>
      </tbody>
    </table>
    <div class="d-flex justify-content-evenly">
      <Paginate
        v-model="page"
        :page-count="pageCount"
        :page-range="5"
        :margin-pages="5"
        :prev-text="'<<'"
        :next-text="'>>'"
        :container-class="'pagination'"
        :page-class="'page-item'"
        :first-last-button="true"
      >
      </Paginate>
      <div>
        <label for="range-select">顯示</label>
        <select v-model="pageRange" id="range-select">
          <option v-for="range in pageRangeList" :key="range" :value="range">{{ range }}筆</option>
        </select>
      </div>
    </div>
  </div>
</template>

<style scoped>
.td {
  width: 8%;
}
</style>
