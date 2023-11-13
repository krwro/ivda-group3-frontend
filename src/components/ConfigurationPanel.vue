<template>
  <div>
    <v-container fluid>
      <v-row>
        <v-col cols="12" md="2" class="sideBar">
            <v-row>
              <v-col cols="12" sm="12">
                <div class="control-panel-font">Company Overview</div>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="12" sm="12">
                <v-select
                    :items="categories.values"
                    label="Select a category"
                    dense
                    v-model="categories.selectedValue"
                    @change="changeCategory"
                ></v-select>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="12" sm="12">
                <div class="control-panel-font">Profit View of Company</div>
              </v-col>
            </v-row>
          <v-row>
            <v-col cols="12" sm="12">
              <v-select
                  :items="toRaw(companies.companyList)"
                  item-value="id"
                  item-title="name"
                  label="Select a company"
                  dense
                  return-object
                  v-model="companies.selectedValue"
                  @change="changeCompany"
              ></v-select>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" sm="12">
              <v-select
                  :items="algorithm.values"
                  label="Select an algorithm for prediction"
                  dense
                  v-model="algorithm.selectedValue"
                  @change="changeAlgorithm"
              ></v-select>
            </v-col>
          </v-row>
        </v-col>
        <v-col cols="12" md="4">
          <ScatterPlot :key="scatterPlotId"
                       :selectedCategory="categories.selectedValue"
                       @changeCurrentlySelectedCompany="changeCurrentlySelectedCompany"
          />
        </v-col>
        <v-col cols="12" md="3">
          <LinePlot :key="linePlotId"
                    :selectedCompany="companies.selectedValue"
                    :selectedCompanyName = "companies.selectedName"
                    :selectedAlgorithm="algorithm.selectedValue"/>
        </v-col>
        <v-col cols="12" md="3">
          <BarChart :key="barChartId"
                    :selectedCompany="companies.selectedValue"/>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>
import ScatterPlot from './ScatterPlot';
import LinePlot from './LinePlot';
import {toRaw} from "vue";
import BarChart from "@/components/BarChartEmployees.vue";
export default {
  components: {BarChart, ScatterPlot, LinePlot},
  data: () => ({
    scatterPlotId: 0,
    linePlotId: 0,
    barChartId: 0,
    categories: {
      values: ['All', 'tech', 'health', 'bank'],
      selectedValue: 'All'
    },
    companies: {
      selectedValue: {name:"alphabet", id:"1"},
      selectedName: "",
      companyList: []
    },
    algorithm: {
      values: ['none', 'random', 'regression'],
      selectedValue: 'none'
    }
  }),
  methods: {
    toRaw,
    changeCategory() {
      this.scatterPlotId += 1
    },
    changeCompany() {
      console.log(this.companies.selectedValue)
      this.linePlotId += 1
      this.barChartId +=1
    },
    changeAlgorithm() {
      this.linePlotId += 1
    },
    changeCurrentlySelectedCompany(companyId) {
      this.companies.selectedValue = this.companies.companyList[companyId-1]
      this.changeCompany()
    },
    async fetchData() {
      class Company {
        constructor(id, name) {
          this.id = id;
          this.name = name;
        }
      }
      // req URL to retrieve companies from backend
      var reqUrl = 'http://127.0.0.1:5000/companies?category=' + this.categories.selectedValue
      console.log("ReqURL " + reqUrl)
      // await response and data
      const response = await fetch(reqUrl)
      const responseData = await response.json();
      // transform data to usable by scatterplot
      const companyArray = []
      responseData.forEach((company) => {
        companyArray.push(new Company(company.id, company.name))
        this.companies.companyList.push(new Company(company.id, company.name))
      })
      this.companies.companyList = companyArray
      this.companies.selectedValue = companyArray[0]
      this.companies.companyList = toRaw(this.companies.companyList)
    },
  },
  mounted() {
    this.fetchData()
  }
}
</script>

<style scoped>
.control-panel-font {
  font-family: "Open Sans", verdana, arial, sans-serif;
  align-items: center;
  font-size: 15px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  display: flex;
  font-weight: 500;
  height: 40px;
}
.sideBar {
  border-right: 1px solid rgba(0, 0, 0, 0.1);
  background: #dbdeff;
  padding-left: 17px;
  height: calc(100vh - 50px);
}
</style>
