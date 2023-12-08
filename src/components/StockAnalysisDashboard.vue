<template>
  <div>
    <v-container :fluid="true">
      <v-row>
        <v-col cols="12" md="3">
          <RankingPanel @ranking-complete="updateRankedStocks" @date-range-updated="updateDateRange" @selected-features="updateSelectedFeatures"/>
        </v-col>
        <v-col cols="12" md="3">
          <h1>Ranked Stocks</h1>
          <p>
            Here the ranked stocks are displayed alongside their average score and average financial metrics over the selected timeframe.
            <span class="text-link" @click="dialog = true">
              How is the score calculated?
            </span>
          </p>
            <v-dialog v-model="dialog" max-width="600px">
              <v-card>
                <v-card-title>
                  Score Calculation Formula
                </v-card-title>
                <v-card-text>
                  <p>The score for each stock is calculated using a combination of weighted features and a decay factor. Here's a detailed breakdown:</p>
                  <ul>
                    <li><strong>Feature Selection:</strong> Selected financial metrics (features) are used in the score calculation. These might include metrics like revenue, profit margins, etc.</li>
                    <li><strong>Normalization:</strong> Each feature is normalized using Min-Max scaling to ensure a consistent range between 0 and 1. This step is crucial for comparing features on the same scale.</li>
                    <li><strong>Weighting:</strong> Each normalized feature is multiplied by a predetermined weight. The weight reflects the relative importance of the feature in the overall score.</li>
                    <li><strong>Decay Factor:</strong> A decay factor is applied to account for the timeliness of the data. More recent data is often more relevant, so older data is 'decayed' to reduce its impact. The decay can be linear, exponential, or logarithmic.</li>
                    <li><strong>Score Calculation:</strong> The final score for each stock is calculated by summing the weighted, decay-adjusted scores of each feature. Mathematically, it's represented as: <br/>
                      Score<sub>i</sub> = Σ<sub>f ∈ F</sub> (s<sub>i,f</sub> · w<sub>f</sub> · d<sub>i</sub>) <br/>
                      Where F is the set of selected features,<br> s<sub>i,f</sub> is the normalized score for feature f of stock i,<br> w<sub>f</sub> is the weight for feature f,<br> d<sub>i</sub> is the decay factor for stock i.</li>
                  </ul>
                  <p>This methodology ensures a fair and dynamic ranking of stocks based on up-to-date and relevant financial metrics.</p>

                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text="true" @click="dialog = false">Close</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          <StockTable :data="rankedStocks" @update-selected-stocks="updateSelectedStocks"/>
        </v-col>
        <v-col cols="12" md="6">
          <v-tabs v-model="tab" background-color="transparent">
            <v-tab value="timeSeries">Time Series</v-tab>
            <v-tab value="histogram">Histogram</v-tab>
            <v-tab value="scatterMatrix">Scatter Matrix</v-tab>
          </v-tabs>

          <v-card-text>
            <v-window v-model="tab">
              <v-window-item value="timeSeries">
                <StockTimeSeriesPlots :selectedStocks="selectedStocks" :date-range="dateRange" :selectedFeatures="selectedFeatures"/>

              </v-window-item>

              <v-window-item value="histogram">
                <HistogramGrid :startDate="dateRange[0]" :endDate="dateRange[1]"/>
              </v-window-item>

              <v-window-item value="scatterMatrix">
                <ScatterMatrix :rankingData="rankedStocks" :selectedStocks="selectedStocks"/>
              </v-window-item>
            </v-window>
          </v-card-text>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import StockTable from "@/components/StockTable.vue";
import RankingPanel from "@/components/RankingPanel.vue";
import StockTimeSeriesPlots from "@/components/StockTimeSeriesPlots.vue";
import ScatterMatrix from "@/components/ScatterMatrix.vue";
import HistogramGrid from "@/components/HistogramGrid.vue";

export default {
  components: {ScatterMatrix, StockTimeSeriesPlots, StockTable, RankingPanel, HistogramGrid},
  data() {
    return {
      dialog: false,
      columnNames: [],
      selectedAttribute: 'price',
      rankedStocks: [],
      selectedStocks: [],
      dateRange: [],
      selectedFeatures: [],
      tab: null,
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

<style>
.text-link {
  color: #1976d2;
  cursor: pointer;
  text-decoration: underline;
}
</style>
