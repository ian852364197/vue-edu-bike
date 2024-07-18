<script setup>
import { computed, onMounted, ref } from 'vue';
import Paginate from 'vuejs-paginate-next';

const bikeList = ref([]);

const page = ref();

const pageRangeList = ref([10, 20]);
const pageRange = ref(pageRangeList.value[0]);
const pageCount = computed(() => {
  return parseInt(bikeList.value.length / pageRange.value) + 1;
});

onMounted(async () => {
  const test = await fetch(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );
  test.json().then((value) => {
    bikeList.value = value;
    console.log(bikeList.value[0]);
  });
});
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
        <th scope="col">可租借的腳踏車數量</th>
        <th scope="col">站點緯度</th>
        <th scope="col">站點經度</th>
        <th scope="col">可歸還的腳踏車數量</th>
      </tr>
    </thead>
    <tbody>
      <!-- <tr v-for="station in bikeList" :key="station.sno">
        <th scope="row">{{ station.sno }}</th>
        <td>{{ station.sna }}</td>
        <td>{{ station.sarea }}</td>
        <td>{{ station.ar }}</td>
        <td>{{ station.total }}</td>
        <td>{{ station.available_rent_bikes }}</td>
        <td>{{ station.latitude }}</td>
        <td>{{ station.longitude }}</td>
        <td>{{ station.available_return_bikes }}</td>
      </tr> -->
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
