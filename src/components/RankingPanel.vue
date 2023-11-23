<template>
  <div class="feature-ranking-panel">
    <h1>Feature Ranking Panel</h1>
    <p>First choose the features you want to include in your Ranking. Then choose their ranking weights based on their
      importance</p>
    <div class="scrollable-section">
      <v-range-slider
          v-model="dateRange"
          :max="maxDate"
          :min="minDate"
          :step="1"
          label="Year"
          thumb-label="always"
          @end="emitDateRange, rankStocks"
      ></v-range-slider>
        <v-select
            v-model="decayFunction"
            :items="decayFunctions"
            label="Decay Function Type"
            @change="rankStocks"
        ></v-select>
        <v-slider
            v-model="decayRate"
            :max="100"
            :min="0"
            :step="1"
            label="Decay Rate"
            thumb-label="always"
            tick-size="4"
            @end="rankStocks"
        ></v-slider>
      <div v-for="feature in features" :key="feature">
        <v-row>
          <v-checkbox v-model="featureStates[feature]" :label="feature" @change="rankStocks"></v-checkbox>
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
    <!-- <v-btn class="rank-stocks-button" @click="rankStocks">
      RANK STOCKS
    </v-btn> -->
  </div>
</template>


<script>
export default {
  data() {
    return {
      dateRange: [2010, 2022],
      minDate: null,
      maxDate: null,
      features: ['price', 'revenue'],
      featureStates: {'price':true, 'revenue':true},
      featureValues: {'price':50, 'revenue':100},
      tickLabels: {
        0: 'Not Important',
        25: '',
        50: '',
        75: '',
        100: 'Very Important',
      },
      decayRate: 0,
      decayFunction: 'linear',
      decayFunctions: [
        { title: 'Linear', value: 'linear' },
        { title: 'Exponential', value: 'exponential' },
        { title: 'Logarithmic', value: 'logarithmic' },
      ],
    };
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
    fetchFeatures() {
      fetch('http://127.0.0.1:5000/stock-features')
          .then(response => response.json())
          .then(data => {
            this.features = data.features;
            this.initializeFeatureStates();
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
    initializeFeatureStates() {
      this.features.forEach(feature => {
        this.$set(this.featureStates, feature, false);
        this.$set(this.featureValues, feature, 0);
      });
    },
    emitDateRange() {
      this.$emit('date-range-updated', this.dateRange);
    },
    rankStocks() {
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
.feature-ranking-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 20px;
  box-sizing: border-box;
  background-color: #f4f4f4;
}

.scrollable-section {
  height: 72vh;
  overflow-y: auto;
  flex-grow: 1;
  margin-top: 20px;
  padding: 40px 20px 20px;
}

.rank-stocks-button {
  background-color: #4CAF50;
  color: white;
  margin-top: 20px;
}

.rank-stocks-button:hover {
  background-color: #388E3C;
}
</style>
