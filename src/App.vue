<script>
import axios from 'axios';

export default {
  data() {
    return {
      prefectures: [],
      loading: true,
    };
  },
  mounted() {
    const apiKey = your-api-key;
    // APIリクエスト
    axios.get('https://opendata.resas-portal.go.jp/api/v1/prefectures', {
      headers: {
        'X-API-KEY': apiKey,
      },
    })
      .then(response => {
        console.log(response);  // レスポンスの確認
        return response.data;
      })
      .then(data => {
        console.log(data.result);  // データの確認
        this.prefectures = data.result;
      })
      .catch(error => {
        console.error('APIリクエストエラー:', error);
      })
      .finally(() => {
        this.loading = false;
      });
  },
}
</script>

<template>
  <div>
    <h1>都道府県一覧</h1>
    <div v-if="loading">Loading...</div>
    <div v-else>
      <div v-for="prefecture in prefectures" :key="prefecture.prefCode" >
        <input type="checkbox" :id="'prefecture-' + prefecture.prefCode" />
        <label :for="'prefecture-' + prefecture.prefCode">{{ prefecture.prefName }}</label>
      </div>
    </div>
  </div>
</template>

<style scoped>
h1 {
  text-align: center;
}

header {
  line-height: 1.5;
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
