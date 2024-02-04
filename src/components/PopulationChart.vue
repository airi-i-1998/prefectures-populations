<template>
  <div>
    <button @click="fetchPopulationData">表示</button>
    <div v-if="loading">Loading...</div>
    <div v-else>
      <canvas id="prefChart"></canvas>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { Chart } from 'chart.js/auto';

export default {
  props: {
    selectedPrefectures: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      loading: false,
      chartData: {},
      chartOptions: {},
      prefectures: [], // 都道府県一覧を取得する場合は適切な API を利用して設定
    };
  },
  mounted() {
    // チャートを作成する処理は fetchPopulationData ではなく、mounted で行います
    this.createChart();
  },

  methods: {
    fetchPopulationData() {
      const selectedCodes = this.selectedPrefectures.join(',');
      const apiKey = your - api - key;
      this.loading = true;
      console.log(selectedCodes);

      axios.get("https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?prefCode=${selectedCodes}", {
        headers: {
          'X-API-KEY': apiKey,
        },
      })
        .then(response => {
          console.log(response); // レスポンス全体を確認
          console.log(response.data); // データの部分だけを確認
          this.chartData = this.parseChartData(response.data.result);
        })
        .catch(error => {
          console.error('API リクエストエラー:', error);
        })
        .finally(() => {
          this.loading = false;
        });
    },
  },
};
</script>

<style scoped>
</style>
