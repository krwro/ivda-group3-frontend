<template>
  <div class="feature-ranking-panel">
    <h1>Feature Ranking Panel</h1>
    <p>First choose the features you want to include in your Ranking. Then choose their ranking weights based on their
      importance</p>
    <v-range-slider
        v-model="dateRange"
        :max="maxDate"
        :min="minDate"
        :step="1"
        label="Year"
        thumb-label="always"
        @end="emitDateRange"
    ></v-range-slider>
    <v-select
        v-model="decayFunction"
        :items="decayFunctions"
        label="Decay Function Type"
        @update:modelValue="rankStocks"
    ></v-select>
    <v-slider
        v-if="decayFunction !== 'none'"
        v-model="decayRate"
        :max="100"
        :min="0"
        :step="1"
        label="Decay Rate"
        thumb-label="always"
        tick-size="4"
        @end="rankStocks"
    ></v-slider>
    <div class="scrollable-section">
      <div v-for="feature in features" :key="feature">
        <v-row>
          <v-checkbox v-model="featureStates[feature]" :label="featureLabels[feature]" @change="rankStocks"></v-checkbox>
          <v-icon
              small
              class="info-icon"
              @click="openFeatureInfo(feature)"
          >
            mdi-help-circle-outline
          </v-icon>
        </v-row>
        <v-slider
            v-if="featureStates[feature]"
            v-model="featureValues[feature]"
            :max="100"
            :step="25"
            :tick-labels="tickLabels"
            :ticks="tickLabels"
            show-ticks="always"
            thumb-label="always"
            tick-size="4"
            @end="rankStocks"
        ></v-slider>
      </div>
    </div>

    <v-dialog v-model="isLoading" :persistent="true" width="300">
      <v-card class="loading-dialog">
        <v-card-title class="headline">Loading...</v-card-title>
        <v-card-text class="text-center">
          <v-progress-circular indeterminate color="primary" size="70" width="7"></v-progress-circular>
          <div class="loading-text">Please wait, ranking in progress.</div>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="infoDialog" max-width="600px">
      <v-card>
        <v-card-title>{{ currentFeatureLabel }}</v-card-title>
        <v-card-text v-html="currentFeatureInfo"></v-card-text>
        <v-card-actions>
          <v-btn color="primary" :text="true" @click="infoDialog = false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>


<script>
export default {
  data() {
    return {
      dateRange: [2010, 2022],
      minDate: null,
      maxDate: null,
      features: ['price', 'revenue'],
      featureStates: {'price':true, 'revenue':true},
      featureValues: {'price':50, 'revenue':100},
      featureLabels: {},
      tickLabels: {
        0: 'Not Important',
        25: '',
        50: '',
        75: '',
        100: 'Very Important',
      },
      decayRate: 0,
      decayFunction: 'none',
      decayFunctions: [
        { title: 'None', value: 'none' },
        { title: 'Linear', value: 'linear' },
        { title: 'Exponential', value: 'exponential' },
        { title: 'Logarithmic', value: 'logarithmic' },
      ],
      isLoading: false,
      infoDialog: false,
      currentFeature: '',
      featureInfo: {
        'price': 'The current market price of the stock. <a href="https://www.investopedia.com/articles/stocks/08/stock-prices-fool.asp" target="_blank">Learn more</a>',
        'F1_price': 'The growth rate of the stock price over a specified period. <a href="https://www.investopedia.com/articles/stocks/08/stock-prices-fool.asp" target="_blank">Learn more</a>',
        'F2_price': 'The momentum of the stock price, indicating recent performance trends. <a href="https://www.investopedia.com/articles/stocks/08/stock-prices-fool.asp" target="_blank">Learn more</a>',

        'grossProfitMargin': 'The percentage of revenue that exceeds the cost of goods sold. <a href="https://www.investopedia.com/terms/g/gross_profit_margin.asp" target="_blank">Learn more</a>',
        'eps': 'Earnings per share, indicating the company\'s profitability. <a href="https://www.investopedia.com/terms/e/eps.asp" target="_blank">Learn more</a>',
        'dividendYield': 'The ratio of annual dividends compared to the stock price. <a href="https://www.investopedia.com/terms/d/dividendyield.asp" target="_blank">Learn more</a>',
        'grahamNumber': 'A figure that measures a stock\'s fundamental value. <a href="https://www.investopedia.com/terms/g/graham-number.asp" target="_blank">Learn more</a>',
        'cashFlowToDebtRatio': 'The ratio of operating cash flow to total debt, a measure of financial durability. <a href="https://www.investopedia.com/terms/c/cash-flowtodebt-ratio.asp" target="_blank">Learn more</a>',
        'operatingCashFlowPerShare': 'The amount of cash generated per share from operations. <a href="https://www.investopedia.com/terms/c/cashflowpershare.asp" target="_blank">Learn more</a>',
        'returnOnAssets': 'The percentage of profit relative to the company\'s total assets. <a href="https://www.investopedia.com/terms/r/returnonassets.asp" target="_blank">Learn more</a>',
        'roe': 'Return on equity, measuring profitability relative to shareholder equity. <a href="https://www.investopedia.com/terms/r/returnonequity.asp" target="_blank">Learn more</a>',
        'debtEquityRatio': 'The ratio of total debt to shareholder equity. <a href="https://www.investopedia.com/terms/d/debtequityratio.asp" target="_blank">Learn more</a>',

        'revenue': 'The total income generated from sales of goods or services. <a href="https://www.investopedia.com/terms/r/revenue.asp" target="_blank">Learn more</a>',
        'F1_revenue': 'The growth rate of revenue over a specified period. <a href="https://www.investopedia.com/terms/r/revenue.asp" target="_blank">Learn more</a>',
        'F2_revenue': 'The momentum of revenue, indicating recent performance trends. <a href="https://www.investopedia.com/terms/r/revenue.asp" target="_blank">Learn more</a>',

        'netIncome': 'The company\'s total earnings or profit. <a href="https://www.investopedia.com/terms/n/netincome.asp" target="_blank">Learn more</a>',
        'F1_netIncome': 'The growth rate of net income over a specified period. <a href="https://www.investopedia.com/terms/n/netincome.asp" target="_blank">Learn more</a>',
        'F2_netIncome': 'The momentum of net income, indicating recent performance trends. <a href="https://www.investopedia.com/terms/n/netincome.asp" target="_blank">Learn more</a>',

        'grossProfit': 'The profit a company makes after deducting the costs of producing and selling its products. <a href="https://www.investopedia.com/terms/g/grossprofit.asp" target="_blank">Learn more</a>',
        'F1_grossProfit': 'The growth rate of gross profit over a specified period. <a href="https://www.investopedia.com/terms/g/grossprofit.asp" target="_blank">Learn more</a>',
        'F2_grossProfit': 'The momentum of gross profit, indicating recent performance trends. <a href="https://www.investopedia.com/terms/g/grossprofit.asp" target="_blank">Learn more</a>',

        'interestCoverage': 'A ratio used to determine how easily a company can pay interest on outstanding debt. <a href="https://www.investopedia.com/terms/i/interestcoverageratio.asp" target="_blank">Learn more</a>',
        'F1_interestCoverage': 'The growth rate of interest coverage over a specified period. <a href="https://www.investopedia.com/terms/i/interestcoverageratio.asp" target="_blank">Learn more</a>',
        'F2_interestCoverage': 'The momentum of interest coverage, indicating recent performance trends. <a href="https://www.investopedia.com/terms/i/interestcoverageratio.asp" target="_blank">Learn more</a>',

        'operatingIncome': 'Income earned from normal business operations. <a href="https://www.investopedia.com/terms/o/operatingincome.asp" target="_blank">Learn more</a>',
        'F1_operatingIncome': 'The growth rate of operating income over a specified period. <a href="https://www.investopedia.com/terms/o/operatingincome.asp" target="_blank">Learn more</a>',
        'F2_operatingIncome': 'The momentum of operating income, indicating recent performance trends. <a href="https://www.investopedia.com/terms/o/operatingincome.asp" target="_blank">Learn more</a>',

        'bookValuePerShare': 'The ratio of shareholder equity to the number of shares outstanding. <a href="https://www.investopedia.com/terms/b/bvps.asp" target="_blank">Learn more</a>',
        'F1_bookValuePerShare': 'The growth rate of book value per share over a specified period. <a href="https://www.investopedia.com/terms/b/bvps.asp" target="_blank">Learn more</a>',
        'F2_bookValuePerShare': 'The momentum of book value per share, indicating recent performance trends. <a href="https://www.investopedia.com/terms/b/bvps.asp" target="_blank">Learn more</a>',

        'tangibleAssetValue': 'The value of a company\'s physical assets. <a href="https://www.investopedia.com/terms/t/tangibleasset.asp" target="_blank">Learn more</a>',
        'F1_tangibleAssetValue': 'The growth rate of tangible asset value over a specified period. <a href="https://www.investopedia.com/terms/t/tangibleasset.asp" target="_blank">Learn more</a>',
        'F2_tangibleAssetValue': 'The momentum of tangible asset value, indicating recent performance trends. <a href="https://www.investopedia.com/terms/t/tangibleasset.asp" target="_blank">Learn more</a>',

        'workingCapital': 'The capital of a business used in its day-to-day trading operations. <a href="https://www.investopedia.com/terms/w/workingcapital.asp" target="_blank">Learn more</a>',
        'F1_workingCapital': 'The growth rate of working capital over a specified period. <a href="https://www.investopedia.com/terms/w/workingcapital.asp" target="_blank">Learn more</a>',
        'F2_workingCapital': 'The momentum of working capital, indicating recent performance trends. <a href="https://www.investopedia.com/terms/w/workingcapital.asp" target="_blank">Learn more</a>',

        'priceToSalesRatio': 'A valuation ratio comparing a company’s stock price to its revenues. <a href="https://www.investopedia.com/terms/p/price-to-salesratio.asp" target="_blank">Learn more</a>',
        'F1_priceToSalesRatio': 'The growth rate of the price to sales ratio over a specified period. <a href="https://www.investopedia.com/terms/p/price-to-salesratio.asp" target="_blank">Learn more</a>',
        'F2_priceToSalesRatio': 'The momentum of the price to sales ratio, indicating recent performance trends. <a href="https://www.investopedia.com/terms/p/price-to-salesratio.asp" target="_blank">Learn more</a>',
      },
    };
  },
  computed: {
    currentFeatureLabel() {
      return this.featureLabels[this.currentFeature] || '';
    },
    currentFeatureInfo() {
      return this.featureInfo[this.currentFeature] || '';
    }
  },
  mounted() {
    this.fetchFeatures();
    this.fetchDateRange();
    this.rankStocks();
  },
  watch: {
    featureStates: {
      handler: 'emitSelectedFeatures',
      deep: true
    }
  },
  methods: {
    fetchFeatures() {
      fetch('http://127.0.0.1:5000/stock-features')
          .then(response => response.json())
          .then(data => {
            this.features = data.features;
            this.initializeFeatureStates();
            this.initializeFeatureValues();
            this.initializeFeatureLabels();
          })
          .catch(error => {
            console.error('Error fetching features:', error);
          });
    },
    fetchDateRange() {
      fetch('http://127.0.0.1:5000/date-range')
          .then(response => response.json())
          .then(data => {
            this.minDate = data.min_date;
            this.maxDate = data.max_date;
            this.dateRange = [2010, 2022]
          })
          .catch(error => {
            console.error('Error fetching features:', error);
          });
    },
    initializeFeatureStates() {
      this.features.forEach(feature => {
        // Default value setting
        this.featureStates[feature] = feature === 'price' || feature === 'revenue';
      });
    },
    initializeFeatureValues() {
      // Default value setting
      this.features.forEach(feature => {
        if (feature === 'price') {
          this.featureValues[feature] = 50;
        } else if (feature === 'revenue') {
          this.featureValues[feature] = 100;
        } else {
          this.featureValues[feature] = 0;
        }
      });
    },

    initializeFeatureLabels() {
      this.features.forEach(feature => {
        this.featureLabels[feature] = this.humanReadableFeatureName(feature);
      });
    },
    humanReadableFeatureName(feature) {
      // Mapping feature codes to human-readable names.
      const names = {
        'price': 'Stock Price',
        'F1_price': 'Stock Price Growth',
        'F2_price': 'Stock Price Momentum',

        'grossProfitMargin': 'Gross Profit Margin',
        'eps': 'Earnings Per Share',
        'dividendYield': 'Dividend Yield',
        'grahamNumber': 'Graham Number',
        'cashFlowToDebtRatio': 'Cash Flow to Debt Ratio',
        'operatingCashFlowPerShare': 'Operating Cash Flow Per Share',
        'returnOnAssets': 'Return on Assets',
        'roe': 'Return on Equity',
        'debtEquityRatio': 'Debt to Equity Ratio',

        'revenue': 'Revenue',
        'F1_revenue': 'Revenue Growth',
        'F2_revenue': 'Revenue Momentum',

        'netIncome': 'Net Income',
        'F1_netIncome': 'Net Income Growth',
        'F2_netIncome': 'Net Income Momentum',

        'grossProfit': 'Gross Profit',
        'F1_grossProfit': 'Gross Profit Growth',
        'F2_grossProfit': 'Gross Profit Momentum',

        'interestCoverage': 'Interest Coverage',
        'F1_interestCoverage': 'Interest Coverage Growth',
        'F2_interestCoverage': 'Interest Coverage Momentum',

        'operatingIncome': 'Operating Income',
        'F1_operatingIncome': 'Operating Income Growth',
        'F2_operatingIncome': 'Operating Income Momentum',

        'bookValuePerShare': 'Book Value Per Share',
        'F1_bookValuePerShare': 'Book Value Per Share Growth',
        'F2_bookValuePerShare': 'Book Value Per Share Momentum',

        'tangibleAssetValue': 'Tangible Asset Value',
        'F1_tangibleAssetValue': 'Tangible Asset Value Growth',
        'F2_tangibleAssetValue': 'Tangible Asset Value Momentum',

        'workingCapital': 'Working Capital',
        'F1_workingCapital': 'Working Capital Growth',
        'F2_workingCapital': 'Working Capital Momentum',

        'priceToSalesRatio': 'Price to Sales Ratio',
        'F1_priceToSalesRatio': 'Price to Sales Ratio Growth',
        'F2_priceToSalesRatio': 'Price to Sales Ratio Momentum'
      };
      return names[feature] || feature;
    },
    openFeatureInfo(feature) {
      this.currentFeature = feature;
      this.infoDialog = true;
    },
    emitDateRange() {
      this.rankStocks()
      this.$emit('date-range-updated', this.dateRange);
    },
    rankStocks() {
      this.isLoading = true;
      const selectedFeatures = this.features
          .filter(feature => this.featureStates[feature])
          .map(feature => ({feature, weight: this.featureValues[feature] || 0}));

      const startDate = `${this.dateRange[0]}-01-01`;
      const endDate = `${this.dateRange[1]}-12-31`;

      fetch('http://127.0.0.1:5000/rank-stocks', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          selectedFeatures,
          startDate,
          endDate,
          decayRate: this.decayRate,
          decayFunction: this.decayFunction
        })
      })
          .then(response => response.json())
          .then(data => {
            this.$emit('ranking-complete', data);
            this.isLoading = false;
          })
          .catch(error => {
            console.error('Error ranking stocks:', error);
          });
      this.$emit('selected-features', selectedFeatures.map(f => f.feature));
    }
  }
}
</script>

<style scoped>
.info-icon {
  padding: 28px;
  cursor: pointer;
}

.feature-ranking-panel {
  display: flex;
  flex-direction: column;
  height: 95vh;
  padding: 20px;
  box-sizing: border-box;
  background-color: #f4f4f4;
}

.scrollable-section {
  overflow-y: auto;
  flex-grow: 1;
  padding: 20px;
}

.loading-dialog {
  border-radius: 10px;
  background-color: #ffffff;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.loading-text {
  margin-top: 20px;
  font-size: 16px;
  color: #555555;
}

.text-center {
  text-align: center;
}

</style>
