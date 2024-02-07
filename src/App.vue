<script>
import PrefectureList from './components/PrefectureList.vue';
import PopulationChart from './components/PopulationChart.vue';

export default {
  data() {
    return {
      prefectures: [],
      selectedPrefectures: [],
      selectedPrefectureNames: [],
    };
  },
  components: { PrefectureList, PopulationChart },
  methods: {
    handleSelectedPrefecturesChange(selectedPrefectures) {
      this.selectedPrefectures = selectedPrefectures;
    },
  },
}
</script>

<template>
  <h1>都道府県別の総人口推移グラフ</h1>
  <PrefectureList :prefectures="prefectures" @selectedPrefecturesChange="handleSelectedPrefecturesChange" />
  <PopulationChart :selectedPrefectures="selectedPrefectures" :selectedPrefectureNames="selectedPrefectureNames" />
</template>

<style scoped>
h1 {
  text-align: center;
}
</style>
