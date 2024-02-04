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
    this.createChart();
  },
  methods: {
    fetchPopulationData() {
      const selectedCodes = this.selectedPrefectures.join(',');
      const apiKey = your-api-key;
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
    createChart() {
      const ctx = document.getElementById("prefChart");
      console.log(ctx);

      var prefChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: this.generateLabels(),
        },
        options: {
          title: {
            display: true,
            text: "人口構成",
          },
          scales: {
            x: {
              type: 'category',
              labels: ['1960年','1965年','1970年','1975年','1980年','1985年','1990年','1995年','2000年','2005年','2010年','2015年','2020年','2025年','2030年','2035年','2040年','2045年'],
            },
            y: {
              suggestedMax: 12000000,
              suggestedMin: 0,
              stepSize: 1000000,
              callback: function (value, index, values) {
                return value + "人";
              },
            },
          },
        },
      });
    },
  },
};
</script>

<style scoped>
</style>
