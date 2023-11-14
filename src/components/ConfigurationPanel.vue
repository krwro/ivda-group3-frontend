<template>
  <div>
    <v-container fluid>
      <v-row>
        <v-col cols="12" md="2">
          <v-select
              v-model="selectedAttribute"
              :items="columnNames"
              label="Select a metric to plot over time"
          ></v-select>
        </v-col>
        <v-col cols="12" md="5">
          <StockChart :selected-attribute="selectedAttribute" />
        </v-col>
        <v-col cols="12" md="5">
          <StockTable/>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import StockChart from "@/components/StockChart.vue";
import StockTable from "@/components/StockTable.vue";

export default {
  components: { StockChart, StockTable },
  data() {
    return {
      columnNames: [],
      selectedAttribute: 'price'
    };
  },
  mounted() {
    this.fetchColumnNames();
  },
  methods: {
    fetchColumnNames() {
      fetch('http://127.0.0.1:5000/stock-features') // Updated endpoint
          .then(response => response.json())
          .then(data => {
            this.columnNames = data.features; // Directly use the returned data
          })
          .catch(error => {
            console.error('Error fetching data:', error);
          });
    }
  }
}
</script>

