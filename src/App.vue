<script setup>
import { computed, onMounted, ref, watch } from 'vue';
import Paginate from 'vuejs-paginate-next';

const bikeList = ref([]);

const page = ref(1);

const pageRangeList = ref([10, 20]);
const pageRange = ref(pageRangeList.value[0]);
const pageCount = computed(() => {
  return parseInt(bikeList.value.length / pageRange.value) + 1;
});

const rentSort = ref(null);
const returnSort = ref(null);

const rentIcon = computed(() => {
  if (!rentSort.value) {
    return '△';
  }
  if (rentSort.value === 'desc') {
    return '▲';
  } else {
    return '▼';
  }
});

const returnIcon = computed(() => {
  if (!returnSort.value) {
    return '△';
  }
  if (returnSort.value === 'desc') {
    return '▲';
  } else {
    return '▼';
  }
});

const filterList = ref([]);

const showList = computed(() => {
  let first = (page.value - 1) * pageRange.value;
  let last = page.value * pageRange.value;
  return filterList.value.slice(first, last);
});

function sortByRent() {
  returnSort.value = null;
  if (rentSort.value === 'desc') {
    rentSort.value = 'asc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.available_rent_bikes - b.available_rent_bikes;
    });
  } else {
    rentSort.value = 'desc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.available_rent_bikes * -1 - b.available_rent_bikes * -1;
    });
  }
}

function sortByReturn() {
  rentSort.value = null;
  if (returnSort.value === 'desc') {
    returnSort.value = 'asc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.available_return_bikes - b.available_return_bikes;
    });
  } else {
    returnSort.value = 'desc';
    filterList.value = filterList.value.sort((a, b) => {
      return a.available_return_bikes * -1 - b.available_return_bikes * -1;
    });
  }
}

onMounted(async () => {
  const test = await fetch(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  test.json().then((value) => {
    bikeList.value = value;
    filterList.value = value;
  });
});

watch(
  () => pageRange.value,
  () => {
    let now = pageRange.value === 10 ? page.value * 2 : parseInt(page.value / 2);
    page.value = now;
  },
  { immediate: true }
);
</script>

<template>
  <h1 class="text-primary">testing</h1>
  <table class="table table-striped">
    <thead>
      <tr>
        <th scope="col">站點編號</th>
        <th scope="col">站點名稱</th>
        <th scope="col">站點所在區域</th>
        <th scope="col">站點地址</th>
        <th scope="col">總車位數量</th>
        <th scope="col" @click="sortByRent">可租借的腳踏車數量 {{ rentIcon }}</th>
        <th scope="col">站點緯度</th>
        <th scope="col">站點經度</th>
        <th scope="col" @click="sortByReturn">可歸還的腳踏車數量 {{ returnIcon }}</th>
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
  <Paginate
    v-model="page"
    :page-count="pageCount"
    :page-range="3"
    :margin-pages="5"
    :prev-text="'<<'"
    :next-text="'>>'"
    :container-class="'pagination'"
    :page-class="'page-item'"
    :first-last-button="true"
  >
  </Paginate>
  <label for="range-select">顯示</label>
  <select v-model="pageRange" id="range-select">
    <option v-for="range in pageRangeList" :key="range" :value="range">{{ range }}筆</option>
  </select>
</template>

<style scoped></style>
