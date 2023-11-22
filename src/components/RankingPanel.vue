<template>
  <div>
    <h1>Feature Ranking Panel</h1>
    <p>First choose the features you want to include in your Ranking. Then choose their ranking weights based on their importance</p>
    <v-range-slider
        v-model="dateRange"
        :min="minDate"
        :max="maxDate"
        :step="1"
        thumb-label="always"
        label="Year"
        @end="emitDateRange"
    ></v-range-slider>
    <div v-for="feature in features" :key="feature">
      <v-row>
        <v-checkbox v-model="featureStates[feature]" :label="feature"></v-checkbox>
      </v-row>
      <v-slider
          v-if="featureStates[feature]"
          v-model="featureValues[feature]"
          show-ticks="always"
          :max="100"
          :step="25"
          :ticks="tickLabels"
          tick-size="4"
          :tick-labels="tickLabels"
          thumb-label="always"
      ></v-slider>
    </div>
    <v-btn @click="rankStocks">
      RANK STOCKS
    </v-btn>
  </div>
</template>


<script>
export default {
  data() {
    return {
      dateRange: [],
      minDate: null,
      maxDate: null,
      features: [],
      featureStates: {},
      featureValues: {},
      tickLabels: {
        0: 'Not Important',
        25: '',
        50: '',
        75: '',
        100: 'Very Important',
      },
    };
  },
  mounted() {
    this.fetchFeatures();
    this.fetchDateRange();
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
          .map(feature => ({ feature, weight: this.featureValues[feature] || 0 }));

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
          endDate
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
};
</script>
