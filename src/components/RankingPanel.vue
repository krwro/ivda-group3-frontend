<template>
  <div>
    <h1>Feature Ranking Panel</h1>
    <p>First choose the features you want to include in your Ranking. Then choose their ranking weights based on their importance</p>
    <v-btn @click="rankStocks">
      RANK STOCKS
    </v-btn>
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
  </div>
</template>


<script>
export default {
  data() {
    return {
      features: [], // Feature labels from backend
      featureStates: {}, // To track the state (checked/unchecked) of each feature
      featureValues: {}, // To store the slider values for each feature
      tickLabels: {
        0: 'Not Important',
        25: '',
        50: '',
        75: '',
        100: 'Very Important',
      }
    };
  },
  mounted() {
    this.fetchFeatures();
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
    initializeFeatureStates() {
      this.features.forEach(feature => {
        this.$set(this.featureStates, feature, false);
        this.$set(this.featureValues, feature, 0);
      });
    },
    rankStocks() {
      const selectedFeatures = this.features
          .filter(feature => this.featureStates[feature])
          .map(feature => ({ feature, weight: this.featureValues[feature] || 0 }));

      // Mock API call - replace with your actual API endpoint and logic
      fetch('http://127.0.0.1:5000/rank-stocks', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ selectedFeatures })
      })
          .then(response => response.json())
          .then(data => {
            this.$emit('ranking-complete', data);
          })
          .catch(error => {
            console.error('Error ranking stocks:', error);
          });
    }
  }
};
</script>
