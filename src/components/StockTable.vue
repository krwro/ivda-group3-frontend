<template>
  <v-btn v-if="checkedStocks.length > 0" class="reset-selection-button" @click="resetSelection">Reset Selection</v-btn>
  <!-- Selected Stocks Table -->
  <div class="scrollable-table-container" v-if="selectedStocks.length > 0">
    <table>
      <thead>
      <tr>
        <th></th>
        <th v-for="key in stockDataKeys" :key="key">{{ key }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(stock, index) in selectedStocks" :key="index">
        <td>
          <input type="checkbox"
                :value="stock.symbol"
                v-model="checkedStocks"
                true-value="checked"
                false-value="unchecked">
        </td>
        <td v-for="(value, key) in stock" :key="key">{{ isNumeric(value) ? roundToThreeDecimals(value) : value }}</td>
      </tr>
      </tbody>
    </table>
  </div>
  <!-- All Stocks Table -->
  <div class="scrollable-table-container">
    <table>
      <thead>
      <tr>
        <th></th>
        <th v-for="key in stockDataKeys" :key="key">{{ key }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(stock, index) in filteredData" :key="index">
        <td>
          <input type="checkbox"
                :value="stock.symbol"
                v-model="checkedStocks"
                true-value="checked"
                false-value="unchecked">
        </td>
        <td v-for="(value, key) in stock" :key="key">{{ isNumeric(value) ? roundToThreeDecimals(value) : value }}</td>
      </tr>
      </tbody>
    </table>
  </div>

</template>

<script>
export default {
  props: {
    data: Array
  },
  data() {
    return {
      checkedStocks: [],
      selectedStocks: []
    };
  },
  computed: {
    stockDataKeys() {
      return this.data.length > 0 ? Object.keys(this.data[0]) : [];
    },
    filteredData() {
      return this.data.filter(stock => !this.selectedStocks.some(selected => selected.symbol === stock.symbol));
    }
  },
  watch: {
    data: {
      handler(newData) {
        // Reset selectedStocks and checkedStocks based on the new data
        this.selectedStocks = newData.filter(stock => this.checkedStocks.includes(stock.symbol));
      },
      deep: true,
      immediate: true
    },
    checkedStocks: {
      handler(newVal) {
        this.selectedStocks = this.data.filter(stock => newVal.includes(stock.symbol));
        this.$emit('update-selected-stocks', newVal);
      },
      deep: true
    }
  },
  methods: {
  roundToThreeDecimals(num) {
    return Math.round(num * 1000) / 1000;
  },
    isNumeric(value) {
      return !isNaN(value) && isFinite(value);
    },
  resetSelection() {
    this.selectedStocks = []
    this.checkedStocks = []
  }
  }
};
</script>


<style scoped>

.reset-selection-button {
  margin: 1vmin;
}
.scrollable-table-container {
  max-height: 40vh;
  overflow-y: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

thead {
  background-color: #f2f2f2;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}

tr:hover {
  background-color: #eaeaea;
}
</style>
