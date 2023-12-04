<template>
  <v-btn v-if="checkedStocks.length > 0" class="reset-selection-button" @click="resetSelection">Reset Selection</v-btn>
  <!-- Selected Stocks Table -->
  <div class="combined-container">
    <div v-if="selectedStocks.length > 0" class="scrollable-table-container">
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
            <input v-model="checkedStocks"
                   :value="stock.symbol"
                   false-value="unchecked"
                   true-value="checked"
                   type="checkbox">
          </td>
          <td v-for="(value, key) in stock" :key="key">{{ isNumeric(value) ? formatNumber(value) : value }}</td>
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
            <input v-model="checkedStocks"
                   :value="stock.symbol"
                   false-value="unchecked"
                   true-value="checked"
                   type="checkbox">
          </td>
          <td v-for="(value, key) in stock" :key="key">{{ isNumeric(value) ? formatNumber(value) : value }}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  emits: ['update-selected-stocks'],
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
    formatNumber(num) {
      num = this.roundToThreeDecimals(num)
      return this.formatLargeNumber(num)
    },

    roundToThreeDecimals(num) {
      return Math.round(num * 1000) / 1000;
    },

    formatLargeNumber(num) {
      let absNum = Math.abs(num);
      let sign = num >= 0 ? '' : '-';

      if (absNum >= 1e9) {
        return `${sign}${(absNum / 1e9).toFixed(3)}B`;
      } else if (absNum >= 1e6) {
        return `${sign}${(absNum / 1e6).toFixed(3)}M`;
      }
      return `${sign}${num}`;
    },

    isNumeric(value) {
      return !isNaN(value) && isFinite(value);
    },

    resetSelection() {
      this.selectedStocks = []
      this.checkedStocks = []
    },
  }
};
</script>


<style scoped>

.reset-selection-button {
  margin: 1vmin;
}

.combined-container {
  max-height: 75vh;
  overflow-y: hidden;
}

.scrollable-table-container {
  max-height: 70vh;
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
