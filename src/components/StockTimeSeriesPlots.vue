<template>
  <div class="multi-metric-container">
    <div v-for="metric in metrics" :key="metric" class="metric-chart">
      <div :id="`chart-${metric}`"></div>
    </div>
  </div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  props: {
    selectedStocks: Array
  },
  data() {
    return {
      metrics: [],
      groupedStockData: {}
    };
  },
  mounted() {
    this.initComponent();
  },
  watch: {
    selectedStocks: 'plotAllMetrics'
  },
  methods: {
    async initComponent() {
      try {
        await this.fetchMetrics();
        await this.fetchAndGroupStockData();
      } catch (error) {
        console.error('Initialization Error:', error);
      }
    },

    async fetchMetrics() {
      const response = await fetch('http://127.0.0.1:5000/stock-features');
      const data = await response.json();
      this.metrics = data.features;
    },

    async fetchAndGroupStockData() {
      const response = await fetch('http://127.0.0.1:5000/stocks');
      const stockData = await response.json();
      this.groupStockData(stockData);
    },

    groupStockData(stockData) {
      this.groupedStockData = stockData.reduce((acc, item) => {
        this.metrics.forEach(metric => {
          acc[item.symbol] = acc[item.symbol] || {};
          acc[item.symbol][metric] = acc[item.symbol][metric] || {dates: [], metrics: []};
          acc[item.symbol][metric].dates.push(item.date);
          acc[item.symbol][metric].metrics.push(item[metric]);
        });
        return acc;
      }, {});
    },

    plotAllMetrics() {
      this.metrics.forEach(metric => {
        const filteredData = this.filterStockDataBySelected(metric);
        this.createPlot(metric, filteredData);
      });
    },

    filterStockDataBySelected(metric) {
      return this.selectedStocks.reduce((acc, symbol) => {
        if (this.groupedStockData[symbol] && this.groupedStockData[symbol][metric]) {
          acc[symbol] = this.groupedStockData[symbol][metric];
        }
        return acc;
      }, {});
    },

    createPlot(metric, filteredData) {
      const traces = this.createTraces(filteredData);
      const layout = this.createLayout(metric);
      Plotly.newPlot(`chart-${metric}`, traces, layout);
    },

    createTraces(filteredData) {
      return Object.entries(filteredData).map(([symbol, data]) => {
        const sortedData = this.sortStockData(data);
        return {
          x: sortedData.map(d => d.date),
          y: sortedData.map(d => d.metric),
          type: 'scatter',
          mode: 'lines',
          name: symbol
        };
      });
    },

    sortStockData(data) {
      return data.dates
          .map((date, index) => ({ date: new Date(date), metric: data.metrics[index] }))
          .sort((a, b) => a.date - b.date)
          .map(d => ({ date: d.date.toISOString().substring(0, 10), metric: d.metric }));
    },

    createLayout(metric) {
      return {
        title: `Stock ${metric} over Time`,
        xaxis: { title: 'Time' },
        yaxis: { title: metric }
      };
    }
  }
}
</script>

<style scoped>
.multi-metric-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 16px;
}

.metric-chart {
  width: 100%;
}
</style>
