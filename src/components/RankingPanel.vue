<template>
  <div class="feature-ranking-panel">
    <h1>Feature Ranking Panel</h1>
    <p>First choose the features you want to include in your Ranking. Then choose their ranking weights based on their
      importance</p>
    <v-range-slider
        v-model="dateRange"
        :max="maxDate"
        :min="minDate"
        :step="1"
        label="Year"
        thumb-label="always"
        @end="emitDateRange"
    ></v-range-slider>
    <v-select
        v-model="decayFunction"
        :items="decayFunctions"
        label="Decay Function Type"
        @update:modelValue="rankStocks"
    ></v-select>
    <v-slider
        v-if="decayFunction !== 'none'"
        v-model="decayRate"
        :max="100"
        :min="0"
        :step="1"
        label="Decay Rate"
        thumb-label="always"
        tick-size="4"
        @end="rankStocks"
    ></v-slider>
    <v-select
        v-model="selectedPreset"
        :items="presets"
        item-title="name"
        item-value="values"
        label="Select a Ranking Preset"
        @update:modelValue="applyPreset"
    ></v-select>
    <div class="scrollable-section">
      <div v-for="feature in features" :key="feature">
        <v-row>
          <v-checkbox v-model="featureStates[feature]" :label="featureLabels[feature]" @change="rankStocks"></v-checkbox>
          <v-icon
              small
              class="info-icon"
              @click="openFeatureInfo(feature)"
          >
            mdi-help-circle-outline
          </v-icon>
        </v-row>
        <v-slider
            v-if="featureStates[feature]"
            v-model="featureValues[feature]"
            :max="100"
            :step="25"
            :tick-labels="tickLabels"
            :ticks="tickLabels"
            show-ticks="always"
            thumb-label="always"
            tick-size="4"
            @end="rankStocks"
        ></v-slider>
      </div>
    </div>

    <v-dialog v-model="isLoading" :persistent="true" width="300">
      <v-card class="loading-dialog">
        <v-card-title class="headline">Loading...</v-card-title>
        <v-card-text class="text-center">
          <v-progress-circular indeterminate color="primary" size="70" width="7"></v-progress-circular>
          <div class="loading-text">Please wait, ranking in progress.</div>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="infoDialog" max-width="600px">
      <v-card>
        <v-card-title>{{ currentFeatureLabel }}</v-card-title>
        <v-card-text v-html="currentFeatureInfo"></v-card-text>
        <v-card-actions>
          <v-btn color="primary" :text="true" @click="infoDialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>


<script>
import { FeatureLabels } from './FeatureLabels.js';
import { FeatureInfo } from './FeatureInfo.js';

export default {
  data() {
    return {
      dateRange: [2010, 2022],
      minDate: null,
      maxDate: null,
      features: ['price', 'revenue', 'netIncome',
        'grossProfit', 'roe', 'bookValuePerShare'],
      featureStates: {'price': true, 'revenue': true, 'netIncome': true,
        'grossProfit': true, 'roe': true, 'bookValuePerShare': true,},
      featureValues: {'price': 50, 'revenue': 50, 'netIncome': 50,
        'grossProfit': 50, 'roe': 50, 'bookValuePerShare': 50,},
      featureLabels: FeatureLabels,
      tickLabels: {
        0: 'Not Important',
        25: '',
        50: '',
        75: '',
        100: 'Very Important',
      },
      decayRate: 0,
      decayFunction: 'none',
      decayFunctions: [
        { title: 'None', value: 'none' },
        { title: 'Linear', value: 'linear' },
        { title: 'Exponential', value: 'exponential' },
        { title: 'Logarithmic', value: 'logarithmic' },
      ],
      isLoading: false,
      infoDialog: false,
      currentFeature: '',
      featureInfo: FeatureInfo,
      selectedPreset: {
        name: 'Balanced',
        values: {
          'price': 50, 'revenue': 50, 'netIncome': 50,
          'grossProfit': 50, 'roe': 50, 'bookValuePerShare': 50,
        },
      },
      presets: [
        {
          name: 'Growth-Oriented',
          values: {
            'F1_price': 75, 'F1_revenue': 75, 'F1_netIncome': 50,
            'F1_grossProfit': 50, 'F1_bookValuePerShare': 50, 'F1_operatingIncome': 50,
          },
        },
        {
          name: 'Value-Oriented',
          values: {
            'priceToSalesRatio': 75, 'dividendYield': 75, 'cashFlowToDebtRatio': 50,
            'grahamNumber': 50, 'roe': 50, 'debtEquityRatio': 50,
          },
        },
        {
          name: 'Momentum-Oriented',
          values: {
            'F2_price': 75, 'F2_revenue': 75, 'F2_netIncome': 50,
            'F2_grossProfit': 50, 'F2_operatingIncome': 50, 'F2_bookValuePerShare': 50,
          },
        },
        {
          name: 'Stability-Oriented',
          values: {
            'eps': 75, 'grossProfitMargin': 75, 'returnOnAssets': 50,
            'interestCoverage': 50, 'workingCapital': 50, 'operatingCashFlowPerShare': 50,
          },
        },
        {
          name: 'Balanced',
          values: {
            'price': 50, 'revenue': 50, 'netIncome': 50,
            'grossProfit': 50, 'roe': 50, 'bookValuePerShare': 50,
          },
        },
        {
          name: 'Custom',
          values: {
          },
        },
      ],
    };
  },
  computed: {
    currentFeatureLabel() {
      return this.featureLabels[this.currentFeature] || '';
    },
    currentFeatureInfo() {
      return this.featureInfo[this.currentFeature] || '';
    }
  },
  mounted() {
    this.fetchFeatures();
    this.fetchDateRange();
    this.rankStocks();
  },
  watch: {
    featureStates: {
      handler: 'emitSelectedFeatures',
      deep: true
    }
  },
  methods: {
    applyPreset() {
      if (this.selectedPreset) {
        // Reset all feature values and states to defaults
        this.features.forEach(feature => {
          this.featureValues[feature] = 0;
          this.featureStates[feature] = false;
        });

        // Apply values from the selected preset
        Object.entries(this.selectedPreset).forEach(([feature, value]) => {
          this.featureValues[feature] = value;
          this.featureStates[feature] = true;
        });

        if (Object.values(this.featureStates).every(value => value === false)) {
          return;
        }
        this.rankStocks();
      }
    },
    fetchFeatures() {
      fetch('http://127.0.0.1:5000/stock-features')
          .then(response => response.json())
          .then(data => {
            this.features = data.features;
            this.initializeFeatureStates();
            this.initializeFeatureValues();
          })
          .catch(error => {
            console.error('Error fetching features:', error);
          });
    },
    fetchDateRange() {
      fetch('http://127.0.0.1:5000/date-range')
          .then(response => response.json())
          .then(data => {
            this.minDate = data.min_date;
            this.maxDate = data.max_date;
            this.dateRange = [2010, 2022]
          })
          .catch(error => {
            console.error('Error fetching features:', error);
          });
    },
    // initializeFeatureStates() {
    //   this.features.forEach(feature => {
    //     // Default value setting
    //     this.featureStates[feature] = feature === 'price' || feature === 'revenue';
    //   });
    // },
    // initializeFeatureValues() {
    //   // Default value setting
    //   this.features.forEach(feature => {
    //     if (feature === 'price') {
    //       this.featureValues[feature] = 50;
    //     } else if (feature === 'revenue') {
    //       this.featureValues[feature] = 100;
    //     } else {
    //       this.featureValues[feature] = 0;
    //     }
    //   });
    // },
    openFeatureInfo(feature) {
      this.currentFeature = feature;
      this.infoDialog = true;
    },
    emitDateRange() {
      this.rankStocks()
      this.$emit('date-range-updated', this.dateRange);
    },
    rankStocks() {
      this.isLoading = true;
      const selectedFeatures = this.features
          .filter(feature => this.featureStates[feature])
          .map(feature => ({feature, weight: this.featureValues[feature] || 0}));

      const startDate = `${this.dateRange[0]}-01-01`;
      const endDate = `${this.dateRange[1]}-12-31`;

      fetch('http://127.0.0.1:5000/rank-stocks', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          selectedFeatures,
          startDate,
          endDate,
          decayRate: this.decayRate,
          decayFunction: this.decayFunction
        })
      })
          .then(response => response.json())
          .then(data => {
            this.$emit('ranking-complete', data);
            this.isLoading = false;
          })
          .catch(error => {
            console.error('Error ranking stocks:', error);
          });
      this.$emit('selected-features', selectedFeatures.map(f => f.feature));
    }
  }
}
</script>

<style scoped>
.info-icon {
  padding: 28px;
  cursor: pointer;
}

.feature-ranking-panel {
  display: flex;
  flex-direction: column;
  height: 95vh;
  padding: 20px;
  box-sizing: border-box;
  background-color: #f4f4f4;
}

.scrollable-section {
  overflow-y: auto;
  flex-grow: 1;
  padding: 20px;
}

.loading-dialog {
  border-radius: 10px;
  background-color: #ffffff;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.loading-text {
  margin-top: 20px;
  font-size: 16px;
  color: #555555;
}

.text-center {
  text-align: center;
}

</style>
