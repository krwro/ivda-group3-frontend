<template>
  <div id="stockTable"></div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  name: 'StockTable',
  mounted() {
    this.fetchStockData();
  },
  methods: {
    fetchStockData() {
      fetch('http://127.0.0.1:5000/stocks')
          .then(response => response.json())
          .then(data => {
            this.createTable(data);
          })
          .catch(error => {
            console.error('Error fetching stock data:', error);
          });
    },

    createTable(stockData) {
      // Process stock data to get the latest price and date for each symbol
      const processedData = this.processStockData(stockData);

      // Create table data for Plotly
      const tableData = [{
        type: 'table',
        header: {
          values: [["Symbol"], ["Latest Price"], ["Latest Date"]],
          align: "center",
          line: {width: 1, color: 'black'},
          fill: {color: "grey"},
          font: {family: "Arial", size: 12, color: "white"}
        },
        cells: {
          values: processedData,
          align: "center",
          line: {color: "black", width: 1},
          fill: {color: ["white"]},
          font: {family: "Arial", size: 11, color: ["black"]}
        }
      }];

      // Plot the table
      Plotly.newPlot('stockTable', tableData);
    },

    processStockData(data) {
      // Group data by symbol
      const groupedData = {};
      data.forEach(item => {
        if (!groupedData[item.symbol]) {
          groupedData[item.symbol] = [];
        }
        groupedData[item.symbol].push({ date: new Date(item.date), price: item.price });
      });

      const symbols = [];
      const latestPrices = [];
      const latestDates = [];

      // Find latest date and price for each symbol
      Object.keys(groupedData).forEach(symbol => {
        const sortedData = groupedData[symbol].sort((a, b) => b.date - a.date); // Sort descending
        const latestEntry = sortedData[0]; // Latest entry

        symbols.push(symbol);
        latestPrices.push(latestEntry.price);
        latestDates.push(latestEntry.date.toISOString().substring(0, 10)); // Format date as string
      });

      return [symbols, latestPrices, latestDates];
    },
  }
}
</script>
