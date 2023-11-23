<template>
  <div>
    <v-container fluid="true">
      <v-row>
        <v-col cols="12" md="3">
          <RankingPanel @ranking-complete="updateRankedStocks" @date-range-updated="updateDateRange" @selected-features="updateSelectedFeatures"/>
        </v-col>
        <v-col cols="12" md="4">
          <h1>Ranked Stocks</h1>
          <p>Here the ranked stocks are displayed alongside their average score and average financial metrics over the selected timeframe</p>
          <StockTable :data="rankedStocks" @update-selected-stocks="updateSelectedStocks"/>
        </v-col>
        <v-col cols="12" md="5">
          <v-select
              v-model="selectedAttribute"
              :items="columnNames"
              label="Select a metric to plot over time"
              v-if="showChart"
          ></v-select>
          <ScatterMatrix :rankingData="rankedStocks" :selectedStocks="selectedStocks"></ScatterMatrix>
          <StockChart :selected-attribute="selectedAttribute" :selected-stocks="selectedStocks"  v-if="showChart"/>
          <StockTimeSeriesPlots :selectedStocks="selectedStocks" :date-range="dateRange" :selectedFeatures="selectedFeatures"/>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import StockChart from "@/components/StockChart.vue";
import StockTable from "@/components/StockTable.vue";
import RankingPanel from "@/components/RankingPanel.vue";
import StockTimeSeriesPlots from "@/components/StockTimeSeriesPlots.vue";
import ScatterMatrix from "@/components/ScatterMatrix.vue";

export default {
  components: {ScatterMatrix, StockTimeSeriesPlots, StockChart, StockTable, RankingPanel },
  data() {
    return {
      columnNames: [],
      selectedAttribute: 'price',
      rankedStocks: [],
      selectedStocks: [],
      dateRange: [],
      showChart: false,
      selectedFeatures: []
    };
  },
  mounted() {
    this.fetchColumnNames();
  },
  methods: {
    fetchColumnNames() {
      fetch('http://127.0.0.1:5000/stock-features')
          .then(response => response.json())
          .then(data => {
            this.columnNames = data.features;
          })
          .catch(error => {
            console.error('Error fetching data:', error);
          });
    },

    updateRankedStocks(response) {
      this.rankedStocks = JSON.parse(response.rankedStocks);
    },

  updateSelectedFeatures(features) {
    this.selectedFeatures = features;
  },

    updateSelectedStocks(selectedStocks) {
      this.selectedStocks = selectedStocks;
    },

    updateDateRange(newRange) {
      this.dateRange = newRange;
    }
  }
}
</script>
