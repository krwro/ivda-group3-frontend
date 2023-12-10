<template>
  <div class="multi-metric-container">
    <div v-for="metric in selectedFeatures" :key="metric" class="metric-chart">
      <div id="multi-metric-subplot"></div>
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
      groupedStockData: {},
      colorMapping: {}
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
  computed: {
    plotHeight() {
      const featureCount = this.selectedFeatures.length;
      const evenFeatureCount = featureCount % 2 === 0 ? featureCount : featureCount + 1;
      const viewHeight = 90
      return evenFeatureCount > 0 ? viewHeight / evenFeatureCount : viewHeight;
    },
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
      this.assignColorsToSymbols()
      const allFilteredData = this.selectedFeatures.map(metric => {
        return {
          metric: metric,
          data: this.filterStockDataBySelected(metric)
        };
      });

      this.createSubplots(allFilteredData);
    },

    createSubplots(allFilteredData) {
      this.$nextTick(() => {
        const subplotId = 'multi-metric-subplot';
        if (document.getElementById(subplotId)) {
          const subplotData = [];
          const n_columns = 2
          const subplotLayout = {
            grid: {rows: allFilteredData.length, columns: n_columns, pattern: 'independent'},
            showlegend: true,
            legend: { x: 1, y: 1 },
            margin: { l: 56, r: 16, t: 16, b: 16 },
            height: window.innerHeight * (this.plotHeight / 100) * allFilteredData.length * n_columns
          };

          allFilteredData.forEach((filteredData, index) => {
            const metricTraces = this.createTraces(filteredData.data, index === 0);
            metricTraces.forEach(trace => {
              trace.xaxis = `x${index + 1}`;
              trace.yaxis = `y${index + 1}`;
              subplotData.push(trace);
            });

            subplotLayout[`yaxis${index + 1}`] = { title: filteredData.metric };
          });

          Plotly.newPlot(subplotId, subplotData, subplotLayout).then(() => {
            this.bindPlotEvents(subplotId, allFilteredData.length);
          });
        } else {
          console.warn(`Element with ID ${subplotId} not found.`);
        }
      });
    },

    bindPlotEvents(subplotId, subplotCount) {
      const plotElement = document.getElementById(subplotId);
      plotElement.on('plotly_relayout', (eventData) => {
        if (eventData['xaxis.range[0]'] && eventData['xaxis.range[1]']) {
          for (let i = 1; i <= subplotCount; i++) {
            Plotly.relayout(subplotId, {
              [`xaxis${i}.range`]: [eventData['xaxis.range[0]'], eventData['xaxis.range[1]']]
            });
          }
        }
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

    generateRandomColor() {
      const h = Math.floor(Math.random() * 360);
      const s = 80; // fixed high saturation for more vivid colors
      const l = Math.floor(Math.random() * 20) + 30; // darker lightness values for better contrast on white
      return `hsl(${h}, ${s}%, ${l}%)`;
    },

    assignColorsToSymbols() {
      this.selectedStocks.forEach(symbol => {
        if (!this.colorMapping[symbol]) {
          let color;
          do {
            color = this.generateRandomColor();
          } while (Object.values(this.colorMapping).includes(color));
          this.colorMapping[symbol] = color;
        }
      });
    },

    createTraces(filteredData, includeInLegend) {
      return Object.entries(filteredData).map(([symbol, data]) => {
        const sortedData = this.sortStockData(data);
        return {
          x: sortedData.map(d => d.date),
          y: sortedData.map(d => d.metric),
          type: 'scatter',
          mode: 'lines',
          name: symbol,
          line: { color: this.colorMapping[symbol] },
          showlegend: includeInLegend // Only show in legend for the first metric
        };
      });
    },


    sortStockData(data) {
      return data.dates
          .map((date, index) => ({ date: new Date(date), metric: data.metrics[index] }))
          .sort((a, b) => a.date - b.date)
          .map(d => ({ date: d.date.toISOString().substring(0, 10), metric: d.metric }));
    }
  }
}
</script>

<style>

.multi-metric-container {
  display: flex;
  flex-direction: column;
  height: 90vh;
  width: 47.5vw;
}

* {
  max-height: 100vh;
}

</style>
