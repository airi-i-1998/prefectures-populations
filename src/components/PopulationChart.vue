<script>
import axios from 'axios';
import { Chart } from 'chart.js';
import 'chart.js/auto';
import { Line } from 'vue-chartjs';

export default {
  extends: Line,
  props: {
    selectedPrefectures: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      loading: false,
      chartData: {
        labels: [],
        datasets: [],
      },
      chartOptions: {
        prefChart: null,
      },
    };
  },
  methods: {
    fetchPopulationData() {
      if (this.prefChart) {
        this.prefChart.destroy();
        this.prefChart = null;
      }
      const apiKey = your-api-key;
      this.loading = true;
      // 選択された都道府県ごとにAPIリクエストを実行
      const requests = this.selectedPrefectures.map(prefCode => {
        return axios.get(`https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?cityCode=-&prefCode=${prefCode}`, {
          headers: {
            'X-API-KEY': apiKey,
          },
        });
      });
      // 全てのAPIリクエストが完了するまで待機
      Promise.all(requests)
        .then(responses => {
          const prefectureData = responses.map(response => {
            if (response && response.data && response.data.result && response.data.result.data) {
              return response.data.result.data
                .filter(item => item.label === '総人口')
                .map(item => ({
                  label: item.label,
                  data: item.data,
                }))[0];
            } else {
              console.error('API レスポンスの構造が不正です。');
              return null;
            }
          });
          // チャートデータを設定
          this.chartData = this.parseChartData(prefectureData);
          this.createChart();
        })
        .catch(error => {
          console.error('API リクエストエラー:', error);
          console.error('Complete Error Object:', JSON.stringify(error, null, 2));
        })
        .finally(() => {
          this.loading = false;
        });
    },
    createChart() {
      if (this.chartData && this.chartData.labels && this.chartData.datasets) {
        const ctx = document.getElementById("prefChart");
        if (ctx) {
          const context = ctx.getContext('2d');
          context.clearRect(0, 0, ctx.width, ctx.height);
          this.prefChart = new Chart(ctx, {
            type: 'line',
            data: {
              labels: this.chartData.labels,
              datasets: this.chartData.datasets,
            },
            options: {
              title: {
                display: true,
              },
              scales: {
                x: {
                  type: 'linear',
                  position: 'bottom',
                  title: {
                    display: true,
                    text: '年',
                  },
                },
                y: {
                  title: {
                    display: true,
                    text: '人口',
                  },
                  angle: 90,
                },
              },
            },
          });
        }
      }
    },
    parseChartData(rawData) {
      if (!rawData || !rawData.length) {
        console.error('Invalid data structure or missing data.');
        return { labels: [], datasets: [] };
      }
      const labels = rawData[0].data ? rawData[0].data.map(entry => entry.year) : [];
      const datasets = rawData.map(item => ({
        label: this.getPrefectureName(item.label),
        data: item.data ? item.data.map(entry => entry.value) : [],
        borderColor: this.getRandomColor(),
        borderWidth: 2,
        fill: false,
      }));
      return {
        labels,
        datasets,
      };
    },
    getRandomColor() {
      const letters = '0123456789ABCDEF';
      let color = '#';
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    },
  },
  mounted() {
    this.fetchPopulationData();
  },
};
</script>

<template>
  <div>
    <button @click="fetchPopulationData">表示</button>
    <div v-if="loading || chartData.labels.length === 0">Loading...</div>
    <div v-else>
      <line-chart :data="chartData" :options="chartOptions"></line-chart>
    </div>
    <canvas id="prefChart"></canvas>
  </div>
</template>

<style scoped>
</style>
