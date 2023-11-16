<template>
  <div id="stockTable"></div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  props: {
    data: Array
  },
  watch: {
    data: {
      immediate: true,
      handler(newData) {
        this.plotTable(newData);
      }
    }
  },
  methods: {
    plotTable(stockData) {
      if (!stockData || stockData.length === 0) return;

      const tableData = [{
        type: 'table',
        header: {
          values: Object.keys(stockData[0]).map(key => [key]),
          align: "center",
          line: { width: 1, color: 'black' },
          fill: { color: "grey" },
          font: { family: "Arial", size: 12, color: "white" }
        },
        cells: {
          values: Object.keys(stockData[0]).map(key => stockData.map(row => row[key])),
          align: "center",
          line: { color: "black", width: 1 },
          fill: { color: ["white"] },
          font: { family: "Arial", size: 11, color: ["black"] }
        }
      }];

      Plotly.newPlot('stockTable', tableData);
    }
  }
};
</script>
