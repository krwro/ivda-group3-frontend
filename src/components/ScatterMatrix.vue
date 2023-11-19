<template>
  <div id="scatterMatrix"></div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  name: 'ScatterMatrix',
  props: {
    rankingData: Array,
    selectedStocks: Array
  },
  watch: {
    rankingData: 'updatePlot',
    selectedStocks: 'updatePlot'
  },
  methods: {
    updatePlot() {
      if (this.rankingData && this.rankingData.length) {
        this.plotScatterMatrix();
      }
    },

    plotScatterMatrix() {
      const traces = this.createTraces();
      const layout = { title: 'Scatter Matrix of Average Metrics' };
      Plotly.newPlot('scatterMatrix', traces, layout);
    },

    createTraces() {
      const selectedData = this.getSelectedData();
      const unselectedData = this.getUnselectedData();

      const selectedTrace = this.createTrace(selectedData, 'rgb(143,0,0)');
      const unselectedTrace = unselectedData.length ? this.createTrace(unselectedData, 'rgba(200, 200, 200, 0.5)') : null;

      return unselectedTrace ? [unselectedTrace, selectedTrace] : [selectedTrace];
    },

    getSelectedData() {
      return this.selectedStocks.length > 0
          ? this.rankingData.filter(row => this.selectedStocks.includes(row.symbol))
          : this.rankingData;
    },

    getUnselectedData() {
      return this.selectedStocks.length > 0
          ? this.rankingData.filter(row => !this.selectedStocks.includes(row.symbol))
          : [];
    },

    createTrace(data, color) {
      const dimensions = this.createDimensions(data);
      return {
        type: 'splom',
        dimensions: dimensions,
        marker: { size: 5, color },
        showlegend: false
      };
    },

    createDimensions(data) {
      const unwantedKeys = ['rank', 'symbol', 'score'];
      return Object.keys(data[0])
          .filter(key => !unwantedKeys.includes(key))
          .map(key => ({ label: key, values: data.map(row => row[key]) }));
    }
  }
}
</script>
