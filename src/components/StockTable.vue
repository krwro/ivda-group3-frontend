<template>
  <div class="scrollable-table-container">
    <table>
      <thead>
      <tr>
        <th></th>
        <th v-for="key in stockDataKeys" :key="key">{{ key }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(stock, index) in data" :key="index">
        <td>
          <input type="checkbox" :value="stock.symbol" v-model="selectedStocks">
        </td>
        <td v-for="(value, key) in stock" :key="key">{{ value }}</td>
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
      selectedStocks: []
    };
  },
  computed: {
    stockDataKeys() {
      return this.data.length > 0 ? Object.keys(this.data[0]) : [];
    }
  },
  watch: {
    selectedStocks: {
      handler(newVal) {
        console.log("hello")
        console.log(newVal)
        this.$emit('update-selected-stocks', newVal);
      },
      deep: true
    }
  }
};
</script>

<style scoped>
.scrollable-table-container {
  max-height: 440px;
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
