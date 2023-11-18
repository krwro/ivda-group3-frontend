<template>
  <div id="stockChart"></div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  name: 'StockChart',
  props: {
    selectedAttribute: String,
    selectedStocks: Array
  },
  data() {
    return{
      stocks: null
    }
  },
  mounted() {
    this.fetchStockData();
  },
  watch: {
    selectedAttribute(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.plotStockData(this.stocks);
      }
    },
    selectedStocks() {
      this.plotStockData(this.stocks);
    }
  },
  methods: {
    fetchStockData() {
      fetch('http://127.0.0.1:5000/stocks')
          .then(response => response.json())
          .then(data => {
            this.plotStockData(data);
            this.stocks = data
          })
          .catch(error => {
            console.error('Error fetching stock data:', error);
          });
    },

    plotStockData(stockData) {
      const groupedData = {};
      stockData.forEach(item => {
        // Check if the current stock is selected
        if (this.selectedStocks.includes(item.symbol)) {
          if (!groupedData[item.symbol]) {
            groupedData[item.symbol] = { dates: [], metrics: [] };
          }
          groupedData[item.symbol].dates.push(item.date);
          const metricValue = item[this.selectedAttribute];
          groupedData[item.symbol].metrics.push(metricValue);
        }
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
