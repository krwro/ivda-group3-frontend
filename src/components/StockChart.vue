<template>
  <div id="stockChart"></div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  name: 'StockChart',
  props: {
    selectedAttribute: String
  },
  mounted() {
    this.fetchStockData();
  },
  watch: {
    selectedAttribute(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.fetchStockData(); // Re-fetch data when selected attribute changes
      }
    }
  },
  methods: {
    fetchStockData() {
      fetch('http://127.0.0.1:5000/stocks')
          .then(response => response.json())
          .then(data => {
            this.plotStockData(data);
          })
          .catch(error => {
            console.error('Error fetching stock data:', error);
          });
    },

    plotStockData(stockData) {
      // Group data by symbol
      const groupedData = {};
      stockData.forEach(item => {
        if (!groupedData[item.symbol]) {
          groupedData[item.symbol] = { dates: [], metrics: [] };
        }
        groupedData[item.symbol].dates.push(item.date);
        const metricValue = item[this.selectedAttribute]
        groupedData[item.symbol].metrics.push(metricValue);
      });

      // Sort data points by date for each symbol and prepare traces
      const traces = Object.keys(groupedData).map(symbol => {
        const sortedData = groupedData[symbol].dates
            .map((date, index) => ({ date: new Date(date), metric: groupedData[symbol].metrics[index] }))
            .sort((a, b) => a.date - b.date);

        const sortedDates = sortedData.map(data => data.date.toISOString().substring(0, 10));
        const sortedMetrics = sortedData.map(data => data.metric);

        return {
          x: sortedDates,
          y: sortedMetrics,
          type: 'scatter',
          mode: 'lines',
          name: symbol
        };
      });

      const layout = {
        title: {
          text: 'Stocks\' ' + this.selectedAttribute + ' over Time',
          font: { family: 'Helvetica', size: 24 }
        },
        xaxis: {
          title: { text: 'Time', font: { family: 'Helvetica', size: 16 } }
        },
        yaxis: {
          title: {
            text: this.selectedAttribute,
            font: { family: 'Helvetica', size: 16 }
          }
        }
      };

      Plotly.newPlot('stockChart', traces, layout);
    },
  }
}
</script>
