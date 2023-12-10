<template>
  <div class="scatter-container">
    <div id="legend-container"></div>
    <div id="scatterMatrix"></div>  
  </div>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';


export default {
  name: 'ScatterMatrix',
  props: {
    rankingData: Array,
    selectedStocks: Array
  },
  data() {
    return {
      colorMapping: {}
    };
  },
  watch: {
    rankingData: 'updatePlot',
    selectedStocks: 'updatePlot'
  },
  methods: {
    updatePlot() {
      if (this.rankingData && this.rankingData.length) {
        this.updateColorMap()
        this.assignColorsToSymbols()
        this.plotScatterMatrix();
        this.createLegend();
      }
    },

    plotScatterMatrix() {
      const traces = this.createTraces();
      const layout = {title: 'Scatter Matrix of Average Metrics'};
      Plotly.newPlot('scatterMatrix', traces, layout);
    },

    createTraces() {
      const selectedData = this.getSelectedData();
      const unselectedData = this.getUnselectedData();

      const symbols = selectedData.map(row => row.symbol)

      const colors = []
      let selectedTrace = this.createTrace(selectedData, 'rgb(143,0,0)');

      if (unselectedData.length > 0) {
        symbols.forEach(symbol => {
          colors.push(this.colorMapping[symbol])
        })
        selectedTrace = this.createTrace(selectedData, colors);
      }

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
      const hoverText = data.map(row => `Symbol: ${row.symbol}`);
 
      return {
        type: 'splom',
        dimensions: dimensions,
        marker: {size: 5, color},
        text: hoverText,
        hoverinfo: 'text',
        showlegend: false
      };
    },

    createDimensions(data) {
      const unwantedKeys = ['rank', 'symbol', 'score'];
      return Object.keys(data[0])
          .filter(key => !unwantedKeys.includes(key))
          .map(key => ({label: key, values: data.map(row => row[key])}));
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

    updateColorMap() {
      for (const [key, value] of Object.entries(this.colorMapping)) {
        console.log(value)
        if(!this.selectedStocks.includes(key)){
          delete this.colorMapping[key]
        }
      }
    },

    createLegend() {
      // Create the custom legend div
      const legendDiv = document.getElementById('legend-container');

      // Clear the existing content of the legend div
      legendDiv.innerHTML = '';

      // Loop through the legend data and create legend items
      console.log(this.colorMapping)
      Object.keys(this.colorMapping).forEach(symbol => {
        const color = this.colorMapping[symbol];

        // Create a legend item element
        const legendItem = document.createElement('div');
        legendItem.classList.add('legend-item');

        // Create a colored box to represent the color
        const legendColor = document.createElement('div');
        legendColor.classList.add('legend-color');
        legendColor.style.backgroundColor = color;

        // Create a text element for the legend item name
        const legendText = document.createElement('span');
        legendText.textContent = symbol;

        // Append the colored box and text to the legend item
        legendItem.appendChild(legendColor);
        legendItem.appendChild(legendText);

        // Append the legend item to the custom legend div
        legendDiv.appendChild(legendItem);
      });
    },
  }
}
</script>

<style>
#scatterMatrix {
  height: 90vh;
  width: 47.5vw;
}

#legend-container {
  margin-top: 1vh;
  display: flex; /* Use flex container */
  justify-content: center;
  align-items: center;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-right: 20px; /* Add some spacing between legend items */
}

.legend-color {
  width: 20px;
  height: 20px;
  margin-right: 5px;
  border: 1px solid #000;
}
</style>
