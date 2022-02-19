<template>
  <div class="container-fluid" v-if="JsonData">
    <div class="pb-5">
      <div class="float-right">
        <b>Sales Period:</b> {{ JsonData.salesPeriod }}
      </div>
    </div>

    <table class="table table-bordered text-center">
      <tbody>
        <tr>
          <th colspan="2"></th>
          <th v-for="singleweek in byWeeks" :key="singleweek.index">
            W{{ singleweek[0].weekNumber }}
          </th>
          <th
            v-for="weatherData in JsonData.weatherData"
            :key="weatherData.index"
          >
            {{ datetomoment(weatherData.wthrDte, "MM/DD") }}
          </th>
        </tr>
        <tr>
          <td colspan="2"></td>
          <td :colspan="byWeeks.length"></td>
          <td
            v-for="weatherData in JsonData.weatherData"
            :key="weatherData.index"
          >
            {{ datetomoment(weatherData.wthrDte, "ddd") }}
          </td>
        </tr>
        <tr>
          <th colspan="2">Category</th>
          <th v-for="singleweek in byWeeks" :key="singleweek.index">
            {{ datetomoment(singleweek[0].fromDate, "MM/DD") }} -
            {{ datetomoment(singleweek[0].toDate, "MM/DD") }}
          </th>
          <td
            v-for="weatherData in JsonData.weatherData"
            :key="weatherData.index"
          >
            {{ weatherData.hiTempNum + "/" + weatherData.loTempNum }}
          </td>
        </tr>
        <tr>
          <th rowspan="2">PSA Total</th>
          <th>Sales</th>
          <td v-for="weeksingle in byWeeks" :key="weeksingle.index">
            {{ weeksingle[0].psaTotalSales }}
          </td>
          <td v-for="t in JsonData.weatherData" :key="t.index"></td>
        </tr>
        <tr>
          <th>WO's</th>
          <td v-for="weeksingle in byWeeks" :key="weeksingle.index">
            {{ weeksingle[0].psaTotalWriteOffs }}
          </td>
          <td v-for="t in JsonData.weatherData" :key="t.index"></td>
        </tr>

        <template
          v-for="(bycode, bycode_index) in JsonDataGroupByCode"
          :key="bycode.index"
        >
          <tr>
            <th rowspan="4">{{ bycode[0]["categoryDescription"] }}</th>
          </tr>
          <tr>
            <th>Del</th>
            <td v-for="single in bycode" :key="single.index">
              {{ single.deliveries }}
            </td>
            <template v-for="t in JsonData.weatherData" :key="t.index">
              <td v-if="(JsonDataGroupByCodeDaily[bycode_index] && JsonDataGroupByCodeDaily[bycode_index][0]['date'] == datetomoment(t.wthrDte, 'Y-MM-DD'))">
                {{ JsonDataGroupByCodeDaily[bycode_index][0].deliveries }}
              </td>
              <td v-else>-</td>
            </template>
          </tr>
          <tr>
            <th>Sales</th>
            <td v-for="single in bycode" :key="single.index">
              {{ single.sales }}
            </td>
            <template v-for="t in JsonData.weatherData" :key="t.index">
              <td v-if="(JsonDataGroupByCodeDaily[bycode_index] && JsonDataGroupByCodeDaily[bycode_index][0]['date'] == datetomoment(t.wthrDte, 'Y-MM-DD'))">
                {{ JsonDataGroupByCodeDaily[bycode_index][0].sales }}
              </td>
              <td v-else>-</td>
            </template>
          </tr>
          <tr>
            <th>WO's</th>
            <td v-for="single in bycode" :key="single.index">
              {{ single.writeOffs }}
            </td>
            <template v-for="t in JsonData.weatherData" :key="t.index">
              <td v-if="(JsonDataGroupByCodeDaily[bycode_index] && JsonDataGroupByCodeDaily[bycode_index][0]['date'] == datetomoment(t.wthrDte, 'Y-MM-DD'))">
                {{ JsonDataGroupByCodeDaily[bycode_index][0].writeOffs }}
              </td>
              <td v-else>-</td>
            </template>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";

export default {
  name: "GridView",
  data() {
    return {
      JsonData: [],
      JsonDataGroupByCode: [],
      JsonDataGroupByCodeDaily: [],
    };
  },
  methods: {
    datetomoment: function (date, format) {
      return moment(date).format(format);
    },
    fetchData() {
      var self = this;
      axios.get("/sample.json").then(function (response) {
        if (response.status == 200) {
          self.JsonData = response.data.response;

          self.JsonDataGroupByCode = self.groupByData(
            self.JsonData.categoryAnalysisWeekly,
            "categoryCode"
          ).reverse();

          self.JsonDataGroupByCodeDaily = self.groupByData(
            self.JsonData.categoryAnalysisDaily,
            "categoryCode"
          ).reverse();

        } else {
          throw new Error("Something wrong to fetch data!");
        }
      });
    },
    groupByData(array, group) {
      var hash = Object.create(null),
        result = [];
      array.forEach(function (a) {
        if (!hash[a[group]]) {
          hash[a[group]] = [];
          result.push(hash[a[group]]);
        }
        hash[a[group]].push(a);
      });
      return result;
    },
  },
  created() {
    this.fetchData();
  },
  computed: {
    byWeeks() {
      return this.groupByData(
        this.JsonData.categoryAnalysisWeekly,
        "weekNumber"
      ).reverse();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import '../assets/bootstrap.min.css';
</style>
