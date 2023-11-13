<template>
  <div>
    <v-row align="center" justify="center" class="mt-1 mb-0">
      <h3>Overview of {{ $props.selectedCategory }} Companies</h3>
    </v-row>
    <div style="height: 90vh">
      <div id='myScatterPlot' style="height: inherit"></div>
    </div>
  </div>
</template>



<script>
import Plotly from 'plotly.js/dist/plotly';
export default {
  name: "ScatterPlot",
  props: [
    "selectedCategory"
  ],
  watch: {
    selectedCategory: function () {
      this.ScatterPlotData.x = [];
      this.ScatterPlotData.y = [];

      this.fetchData();
    }
  },

  data: () => ({
    ScatterPlotData: {x: [], y: [], name: [], category: [], categoryColor: [], id: []},
  }),
  mounted() {
    this.fetchData()
  },
  methods: {
    async fetchData() {
      // req URL to retrieve companies from backend
      var reqUrl = 'http://127.0.0.1:5000/companies?category=' + this.$props.selectedCategory
      console.log("ReqURL " + reqUrl)
      // await response and data
      const response = await fetch(reqUrl)
      const responseData = await response.json();
      // transform data to usable by scatterplot

      this.ScatterPlotData.name = []
      this.ScatterPlotData.x= []
      this.ScatterPlotData.y= []
      this.ScatterPlotData.category= []
      this.ScatterPlotData.id= []

      responseData.forEach((company) => {
        this.ScatterPlotData.name.push(company.name)
        this.ScatterPlotData.x.push(company.founding_year)
        this.ScatterPlotData.y.push(company.employees)
        this.ScatterPlotData.category.push(company.category)
        this.ScatterPlotData.id.push(company.id)
      })
      // after the data is loaded, draw the plot
      this.drawScatterPlot()
    },

    drawScatterPlot() {
      console.log("drawing scatter plot")
      const categories = Array.from(new Set(this.ScatterPlotData.category));

      const traces = []
      for (let category of categories){
        var trace = {x: [],
          y: [],
          mode: 'markers',
          type: 'scatter',
          name: category,
          text: "",
          id: [],
          marker: {
            size: 12,
            color: ""
          }}
        const companyNames = []
        const colorDict = {"tech":"blue", "health":"orange", "bank":"green"}
        for (let i in this.ScatterPlotData.x){
          if (category===this.ScatterPlotData.category[i]){
            trace.x.push(this.ScatterPlotData.x[i])
            trace.y.push(this.ScatterPlotData.y[i])
            trace.id.push(this.ScatterPlotData.id[i])
            trace.marker.color = colorDict[category]
            companyNames.push(this.ScatterPlotData.name[i])
          }
        }
        trace.text = companyNames
        traces.push(trace)
      }

      console.log(traces)
      var data = traces;
      var layout = {
        xaxis: {
          title: {
            text: 'founding year',}
        },
        yaxis: {
          title: {
            text: 'employees',}
        }
      }
      var config = {responsive: true, displayModeBar: false}
      Plotly.newPlot('myScatterPlot', data, layout, config);
      this.clickScatterPlot()
    },

    clickScatterPlot() {
      var pn = 0
      var that = this
      var myPlot = document.getElementById('myScatterPlot')
      myPlot.on('plotly_click', function (data) {
        for (var i = 0; i < data.points.length; i++) {

          // get the index of point
          pn = data.points[i].pointNumber;
          const pid = data.points[i].data.id[pn];
          const curveId = data.points[i].curveNumber
          console.log(data)
          // emit event to change the currently selected company in the a) configuration panel
          // and b) update the Profit View
          that.$emit('changeCurrentlySelectedCompany', pid)

          // revert all colors
          var colors = ['#00000' * 15]

          // and change currently selected color to blue
          colors[pn] = '#3777ee';

          // update the marker and plot
          var update = {'marker': {color: colors, size: 12}};
          Plotly.restyle('myScatterPlot', {'marker': {color: "#000000", size: 12}});
          Plotly.restyle('myScatterPlot', update, curveId);
        }
      });
    }

  }
}
</script>



