<template>
  <div>
    <v-row class="input-section">
      <v-col cols="12" md="4" class="align-self-end">
        <v-range-slider v-model="dateRange" :min="minYear" :max="maxYear" :step="1" label="Year Range" thumb-label="always" @end="fetchHistogramData"/>
      </v-col>
      <v-col cols="12" md="2">
        <v-text-field v-model="numBins" label="Number of Bins" type="number" min="1" @blur="fetchHistogramData" />
      </v-col>
      <v-col cols="12" md="2">
        <v-switch v-model="uniformYAxis" label="Uniform Y-Axis" />
      </v-col>
      <v-col cols="12" md="2">
        <v-switch v-model="removeOutliers" label="Remove Outliers" />
      </v-col>
      <v-col cols="12" md="2">
        <v-select v-model="aggregationMethod" :items="['mean', 'median']" label="Aggregation Method" />
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <div id="all-histograms"></div>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  props: {
    startDate: String,
    endDate: String
  },
  data() {
    return {
      histograms: {},
      numBins: 10,
      removeOutliers: false,
      aggregationMethod: 'mean',
      minYear: 2010,
      maxYear: 2022,
      dateRange: [2010, 2022],
      uniformYAxis: false
    };
  },
  watch: {
    uniformYAxis: 'fetchHistogramData',
    removeOutliers: 'fetchHistogramData',
    aggregationMethod: 'fetchHistogramData'
  },
  mounted() {
    this.fetchDateRange();
    this.fetchHistogramData();
  },
  methods: {
    fetchDateRange() {
      fetch('http://127.0.0.1:5000/date-range')
          .then(response => response.json())
          .then(data => {
            this.minYear = new Date(data.min_date).getFullYear();
            this.maxYear = new Date(data.max_date).getFullYear();
            this.dateRange = [this.minYear, this.maxYear];
          })
          .catch(error => console.error('Error fetching date range:', error));
    },
    fetchHistogramData() {
      const queryParams = new URLSearchParams({
        start_date: `${this.dateRange[0]}-01-01`,
        end_date: `${this.dateRange[1]}-12-31`,
        num_bins: this.numBins,
        remove_outliers: this.removeOutliers,
        aggregation_method: this.aggregationMethod
      });

      fetch(`http://127.0.0.1:5000/feature-distribution?${queryParams.toString()}`)
          .then(response => response.json())
          .then(data => {
            this.histograms = data;
            this.plotHistograms();
          })
          .catch(error => console.error('Error fetching histogram data:', error));
    },
    plotHistograms() {
      this.$nextTick(() => {
        let traces = [];
        let annotations = [];
        let maxYValue = this.uniformYAxis ? Math.max(...Object.values(this.histograms).flatMap(histogramData => histogramData.hist)) : 0;
        let totalFeatures = Object.keys(this.histograms).length;
        let [numRows, numCols] = [Math.ceil(Math.sqrt(totalFeatures)), Math.ceil(totalFeatures / Math.ceil(Math.sqrt(totalFeatures)))];

        Object.entries(this.histograms).forEach(([feature, histogramData], index) => {
          this.addTraceAndAnnotation(traces, annotations, histogramData, feature, index);
        });

        Plotly.newPlot('all-histograms', traces, this.getLayout(numRows, numCols, maxYValue, annotations));
      });
    },
    addTraceAndAnnotation(traces, annotations, histogramData, feature, index) {
      traces.push({
        x: histogramData.bin_edges.slice(0, -1),
        y: histogramData.hist,
        type: 'bar',
        name: feature,
        xaxis: `x${index + 1}`,
        yaxis: `y${index + 1}`
      });

      let [xref, yref] = [`x${index + 1} domain`, `y${index + 1} domain`];

      annotations.push({
        text: feature,
        xref: xref,
        yref: yref,
        x: 0.5,
        y: 1.05,
        xanchor: 'center',
        yanchor: 'bottom',
        showarrow: false
      });
    },
    getLayout(numRows, numCols, maxYValue, annotations) {
      let layout = {
        height: numRows * 300,
        grid: { rows: numRows, columns: numCols, pattern: 'independent', row_heights: new Array(numRows).fill(100) },
        annotations: annotations,
        showlegend: false,
        margin: { l: 10, r: 40, t: 10, b: 10 }
      };

      for (let i = 1; i <= Object.keys(this.histograms).length; i++) {
        layout[`yaxis${i}`] = this.uniformYAxis ? { range: [0, maxYValue] } : {};
      }

      return layout;
    }
  }
}
</script>


<style scoped>
#all-histograms {
  width: 100%;
  height: 80vh;
  overflow-y: auto;
  overflow-x: hidden;
}
.input-section {
  background-color: #f4f4f4;
  padding: 8px;
}
</style>
