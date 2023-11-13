<template>
  <div>
    <v-row align="center" justify="center" class="mt-1 mb-0">
      <h3>Profit View of Company: {{ $props.selectedCompany.name }}</h3>
    </v-row>
    <div style="height: 90vh">
      <div id='myLinePlot' style="height: inherit"></div>
    </div>
  </div>
</template>


<script>
import Plotly from 'plotly.js/dist/plotly';
export default {
  name: "LinePlot",
  props: ["selectedCompany", "selectedAlgorithm", "selectedCompanyName"],
  watch: {
    selectedCompany() {
      this.LinePlotData.x = [];
      this.LinePlotData.y = [];

      this.fetchData();
    },
    selectedAlgorithm() {
      this.LinePlotData.x = [];
      this.LinePlotData.y = [];

      this.fetchData();
    }
  },

  data: () => ({
    LinePlotData: {x: [], y: []}
  }),
  mounted() {
    this.fetchData()
  },
  methods: {
    async fetchData() {
      // req URL to retrieve single company from backend
      var reqUrl = 'http://127.0.0.1:5000/companies/' + this.$props.selectedCompany.id +
          '?algorithm=' + this.$props.selectedAlgorithm
      console.log("ReqURL " + reqUrl)
      // await response and data
      const response = await fetch(reqUrl)
      const responseData = await response.json();
      // transform data to usable by lineplot
      responseData.profit.forEach((profit) => {
        this.LinePlotData.x.push(profit.year)
        this.LinePlotData.y.push(profit.value)
      })
      // draw the lineplot after the data is transformed
      this.drawLinePlot()
    },
    drawLinePlot() {
      const traces = []
      var trace1 = {
        x: this.LinePlotData.x,
        y: this.LinePlotData.y,
        type: 'scatter'
      };

      if (this.selectedAlgorithm !== "none"){
        trace1 = {
          x: this.LinePlotData.x.slice(1),
          y: this.LinePlotData.y.slice(1),
          type: 'scatter',
          name: "actual"
        };

        var tracePrediction = {
          x: this.LinePlotData.x.slice(0,2),
          y: this.LinePlotData.y.slice(0,2),
          type: 'scatter',
          name: "prediction",
          line: {
            dash: 'dot',
            width: 4
          }
        }
        traces.push(tracePrediction)
      }

      traces.unshift(trace1)

      var data = traces;
      var layout = {
        xaxis: {
          title: {
            text: 'year',}
        },
        yaxis: {
          title: {
            text: 'profit',}
        }
      }
      var config = {responsive: true, displayModeBar: false}
      Plotly.newPlot('myLinePlot', data, layout, config);

    }
  }
}
</script>



