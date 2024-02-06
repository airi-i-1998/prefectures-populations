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
      selectedPrefectureNames: [],
    };
  },
  methods: {
    fetchPopulationData() {
      if (this.prefChart) {
        this.prefChart.destroy();
        this.prefChart = null;
      }
      const apiKey =  import.meta.env.VITE_APP_RESAS_API_KEY
      // const apiKey = "50FXDSxFzOObNVDiZW9EBtkJxMWdYjtlrPsP9DH7";
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
                    text: '年度',
                    font: {
                      size: 14,
                      weight: "bold",
                    },
                  },
                  ticks: {
                    stepSize: 5,
                    callback: function (value) {
                      return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, "");
                    },
                  },
                },
                y: {
                  title: {
                    display: true,
                    text: '人口',
                    font: {
                      size: 14,
                      weight: "bold",
                    },
                  },
                },
              },
              plugins: {
                tooltip: {
                  callbacks: {
                    label: function (context) {
                      const value = context.parsed.y;
                      const formattedValue = value.toLocaleString();
                      const year = context.parsed.x;
                      const prefecture = context.dataset.label;
                      return `${year}年 ${prefecture} ${formattedValue}人`;
                    },
                  },
                },
              },
            },
          });
        }
      }
    },
    parseChartData(rawData) {
      if (!rawData || !rawData.length || !rawData[0].data) {
        console.error('Invalid data structure or missing data.');
        return { labels: [], datasets: [] };
      }
      const labels = rawData[0].data.map(entry => entry.year.toString());
      console.log(labels);
      const datasets = rawData.map((item, index) => ({
        label: this.selectedPrefectureNames.length > index ? this.selectedPrefectureNames[index] : "",
        data: item.data.map(entry => entry.value),
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
  watch: {
    selectedPrefectures: {
      handler(newSelectedPrefectures) {
        if (this.prefectures) {
          this.selectedPrefectureNames = newSelectedPrefectures.map(prefCode => {
            const prefecture = this.prefectures.find(pref => pref.prefCode === prefCode);
            return prefecture ? prefecture.prefName : '';
          });
        }
      },
      immediate: true
    }
  },
  mounted() {
    const apiKey =  import.meta.env.VITE_APP_RESAS_API_KEY
    // prefecturesの取得が完了してからfetchPopulationDataを呼び出す
    axios.get('https://opendata.resas-portal.go.jp/api/v1/prefectures', {
      headers: {
        'X-API-KEY': apiKey
      }
    })
      .then(response => {
        console.log(response);
        return response.data;
      })
      .then(data => {
        console.log(data.result);
        console.log('Prefecture data:', data.result);
        this.prefectures = data.result;
        // prefecturesの取得が完了したらfetchPopulationDataを呼び出す
        this.fetchPopulationData();
      })
      .catch(error => {
        console.error('APIリクエストエラー:', error);
      })
      .finally(() => {
        this.loading = false;
      });
  }
};
</script>

<template>
  <div>
    <div class="button-container">
      <button @click="fetchPopulationData" class="button">表示</button>
    </div>
    <div v-if="loading || chartData.labels.length === 0">Loading...</div>
    <div v-else class="chart">
      <line-chart :data="chartData" :options="chartOptions"></line-chart>
    </div>
    <canvas id="prefChart"></canvas>
  </div>
</template>

<style scoped>
.button-container {
  text-align: center;
}

.button {
  margin: 20px auto 0;
  padding: 10px 20px;
  font-size: medium;
}

.chart {
  margin-top: 20px;
}
</style>
