<template>
  <div>
    <v-row align="center" justify="center" class="mt-1 mb-0">
      <h3>Employee Comparison View of Company: {{ $props.selectedCompany.name }}</h3>
    </v-row>
    <div style="height: 90vh">
      <div id='myBarChart' style="height: inherit"></div>
    </div>
  </div>
</template>


<script>
import Plotly from 'plotly.js/dist/plotly';
export default {
  name: "barChart",
  props: ["selectedCompany"],
  watch: {
    selectedCompany() {
      this.BarChartData.x = [];
      this.BarChartData.y = [];

      this.fetchData();
    }
  },

  data: () => ({
    BarChartData: {x: [], y: []}
  }),
  mounted() {
    this.fetchData()
  },
  methods: {
    async fetchData() {
      console.log(this.$props.selectedCompany)
      // req URL to retrieve single company from backend
      var reqUrl = 'http://127.0.0.1:5000/companies/' + this.$props.selectedCompany.id +
          '?algorithm=' + this.$props.selectedAlgorithm
      console.log("ReqURL " + reqUrl)
      // await response and data
      const response = await fetch(reqUrl)
      const responseData = await response.json();
      // transform data to usable by lineplot
      this.BarChartData.x.push(responseData.name)
      this.BarChartData.x.push("average in " + responseData.category + " sector")
      this.BarChartData.x.push("median in " + responseData.category + " sector")
      this.BarChartData.y.push(responseData.employees)
      this.BarChartData.y.push(responseData.average_employees_in_category)
      this.BarChartData.y.push(responseData.median_employees_in_category)
      this.drawBarChart()
    },

    drawBarChart() {
      var trace = {
        x: this.BarChartData.x,
        y: this.BarChartData.y,
        type: 'bar',
        marker: {
          color:["darkblue", "blue", "blue"]
        }
      };
      var data = [trace];
      var layout = {
        yaxis: {
          title: {
            text: 'employees',}
        }
      }
      var config = {responsive: true, displayModeBar: false}
      Plotly.newPlot('myBarChart', data, layout, config);

    }
  }
}
</script>



