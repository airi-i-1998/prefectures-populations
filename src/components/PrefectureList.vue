<script>
import axios from 'axios';

export default {
  props: {
    prefectures: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      prefectures: [],
      loading: true,
      selectedPrefectures: [],
      selectedPrefectureNames: [],
    };
  },
  mounted() {
    const apiKey =  import.meta.env.VITE_APP_RESAS_API_KEY
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
        console.log('Population data:', data.result.data);
        this.prefectures = data.result;
      })
      .catch(error => {
        console.error('APIリクエストエラー:', error);
      })
      .finally(() => {
        this.loading = false;
      });
  },
  watch: {
    selectedPrefectures() {
      this.$emit('selectedPrefecturesChange', this.selectedPrefectures);
    },
  }
}
</script>

<template>
  <div>
    <span class="pref">＜都道府県一覧＞</span>
      <div v-if="loading">Loading...</div>
      <div v-else class="pref-container">
        <div v-for="prefecture in prefectures" :key="prefecture.prefCode" class="pref-item">
          <input type="checkbox" :id="'prefecture-' + prefecture.prefCode" v-model="selectedPrefectures"
            :value="prefecture.prefCode" />
          <label :for="'prefecture-' + prefecture.prefCode">{{ prefecture.prefName }}</label>
        </div>
      </div>
  </div>
</template>

<style scoped>
.pref{
  display: block;
  margin-left: 5%;
  font-size: 20px;
}
.pref-container {
  display: flex;
  flex-wrap: wrap;
  align-items: stretch;
  padding-left: 5%;
}

.pref-item {
  width: calc(100% / 5);
  padding: 10px 20px 0 0;
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
