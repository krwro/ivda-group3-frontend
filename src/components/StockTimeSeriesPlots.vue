<template>
  <div class="multi-metric-container" v-if="selectedStocks.length > 0">
    <div v-for="metric in selectedFeatures" :key="metric" class="metric-chart">
      <div :id="`chart-${metric}`"></div>
    </div>
  </div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  props: {
    selectedStocks: Array,
    selectedFeatures: Array,
    dateRange: Array
  },
  data() {
    return {
      rawStocksData: null,
      groupedStockData: {}
    };
  },
  watch: {
    selectedStocks(newVal) {
      if (newVal && newVal.length > 0) {
        this.processData();
      }
    },
    dateRange() {
      this.processData();
    },
    selectedFeatures() {
      this.processData();
    },
  },
  mounted() {
    this.fetchStockData();
  },
  methods: {
    async fetchStockData() {
      const response = await fetch('http://127.0.0.1:5000/stocks');
      this.rawStocksData = await response.json();
      this.processData();
    },
    processData() {
      if (!this.rawStocksData || this.selectedStocks.length === 0) {
        return;
      }

      let filteredData = this.rawStocksData.filter(item => {
        const itemDate = new Date(item.date);
        const startDate = new Date(`${this.dateRange[0] || 2010}-01-01`);
        const endDate = new Date(`${this.dateRange[1] || 2023}-12-31`);
        return itemDate >= startDate && itemDate <= endDate;
      });

      this.groupStockData(filteredData);
      this.plotAllMetrics();
    },

    groupStockData(filteredData) {
      this.groupedStockData = filteredData.reduce((acc, item) => {
        this.selectedFeatures.forEach(metric => {
          acc[item.symbol] = acc[item.symbol] || {};
          acc[item.symbol][metric] = acc[item.symbol][metric] || {dates: [], metrics: []};
          acc[item.symbol][metric].dates.push(item.date);
          acc[item.symbol][metric].metrics.push(item[metric]);
        });
        return acc;
      }, {});
    },

    plotAllMetrics() {
      if (this.selectedStocks.length === 0) {
        return;
      }
      this.selectedFeatures.forEach(metric => {
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
      this.$nextTick(() => {
        const elementId = `chart-${metric}`;
        if (document.getElementById(elementId)) {
          const traces = this.createTraces(filteredData);
          const layout = this.createLayout(metric);
          Plotly.newPlot(elementId, traces, layout);

          document.getElementById(elementId).on('plotly_relayout', (eventData) => {
            this.onPlotInteraction(metric, eventData);
          });

          document.getElementById(elementId).on('plotly_doubleclick', () => {
            this.onPlotReset();
          });

        } else {
          console.warn(`Element with ID ${elementId} not found.`);
        }
      });
    },

    onPlotInteraction(changedMetric, eventData) {
      if (!eventData['xaxis.range[0]'] || !eventData['xaxis.range[1]']) {
        return;
      }

      this.selectedFeatures.forEach(metric => {
        if (metric !== changedMetric) {
          const elementId = `chart-${metric}`;
          Plotly.relayout(elementId, {
            'xaxis.range': [eventData['xaxis.range[0]'], eventData['xaxis.range[1]']]
          });
        }
      });
      },

    onPlotReset() {
      this.selectedFeatures.forEach(metric => {
        const elementId = `chart-${metric}`;
        Plotly.relayout(elementId, {
          'xaxis.autorange': true,
          'yaxis.autorange': true
        });
      });
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
  height: 45vh;
  overflow-y: auto;
}

.metric-chart {
  width: 100%;
}
</style>
