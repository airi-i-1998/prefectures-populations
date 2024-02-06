<script>
import axios from 'axios';
import PopulationChart from './components/PopulationChart.vue';

export default {
  data() {
    return {
      prefectures: [],
      loading: true,
      selectedPrefectures: [],
      selectedPrefectureNames: [],
    };
  },
  mounted() {
    const apiKey = your-api-key;
    // APIリクエスト
    axios.get('https://opendata.resas-portal.go.jp/api/v1/prefectures', {
      headers: {
        'X-API-KEY': apiKey
      }
    })
      .then(response => {
        console.log(response); // レスポンスの確認
        return response.data;
      })
      .then(data => {
        console.log('Prefecture data:', data.result);
        this.prefectures = data.result;
      })
      .catch(error => {
        console.error('APIリクエストエラー:', error);
      })
      .finally(() => {
        this.loading = false;
      });
  },
  components: { PopulationChart }
}
</script>

<template>
  <div>
    <h1>都道府県別の総人口推移グラフ</h1>
      <span class="pref">＜都道府県一覧＞</span>
      <div v-if="loading">Loading...</div>
      <div v-else class="pref-container">
        <div v-for="prefecture in prefectures" :key="prefecture.prefCode" class="pref-item">
          <input type="checkbox" :id="'prefecture-' + prefecture.prefCode" v-model="selectedPrefectures"
            :value="prefecture.prefCode" />
          <label :for="'prefecture-' + prefecture.prefCode">{{ prefecture.prefName }}</label>
        </div>
      </div>
    <PopulationChart :selectedPrefectures="selectedPrefectures" :selectedPrefectureNames="selectedPrefectureNames" />
  </div>
</template>

<style scoped>
h1{
  text-align: center;
}

header {
  line-height: 1.5;
}

.pref{
  display: block;
  margin-left: 5%;
  font-size: 20px;
}
.pref-container {
  display: flex;
  flex-wrap: wrap;
  text-align: center;
}

.pref-item {
  width: calc(100% / 5);
  padding: 10px 15px 0 0;
  box-sizing: border-box;
  padding-top: 10;
  white-space: nowrap;
  text-overflow: ellipsis;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
