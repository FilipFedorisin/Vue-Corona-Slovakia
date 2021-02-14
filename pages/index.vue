<template>
  <div class="container">
    <table class="regions">
      <tr>
        <th @click="sortRegions('region')">Kraj</th>
        <th @click="sortRegions('totalInfected')">Celkovo nakazený</th>
        <th @click="sortRegions('newInfected')">Nový Nakazený</th>
      </tr>
      <tr v-for="region in regions" :key="region.region">
        <td>{{ region.region }}</td>
        <td>{{ region.totalInfected }}</td>
        <td>{{ region.newInfected }}</td>
      </tr>
    </table>
    <table class="district">
      <tr>
        <th @click="sortDistricts('town')">Okres</th>
        <th @click="sortDistricts('totalInfected')">Celkovo nakazený</th>
        <th @click="sortDistricts('newInfected')">Nový Nakazený</th>
      </tr>
      <tr v-for="district in districts" :key="district.town">
        <td>{{ district.town }}</td>
        <td>{{ district.totalInfected }}</td>
        <td>{{ district.newInfected }}</td>
      </tr>
    </table>
    <line-chart class="lineChart" :chart-data="slovakiaByDate" :options="chartOptions" label="Prípady"></line-chart>
  </div>
</template>

<script>
import axios from 'axios'
import LineChart from '~/components/LineChart.vue'

export default {
  name: 'Index',
  components: {
    LineChart,
  },
  async asyncData() {
    const slovakia = await axios
      .get('https://api.apify.com/v2/key-value-stores/GlTLAdXAuOz6bLAIO/records/LATEST?disableRedirect=true')
      .then((res) => res.data)

    const slovakiaByDate = await axios
      .get('https://covid-193.p.rapidapi.com/history', {
        params: {
          country: 'slovakia',
        },
        headers: {
          'x-rapidapi-key': '63b4e7aa5emsh31ee6582fe06880p1b48e5jsn89fb50032e03',
          'x-rapidapi-host': 'covid-193.p.rapidapi.com',
        },
      })
      .then((res) => {
        const scrapped = []
        res.data.response.forEach((item) => {
          scrapped.push({
            cases: item.cases.total,
            date: item.day,
          })
        })
        return scrapped
      })

    return { slovakia, slovakiaByDate }
  },
  data() {
    return {
      rawRegions: [],
      rawDistricts: [],
      regionsCurrentSort: 'region',
      regionsSortDir: 'asc',
      districtsCurrentSort: 'town',
      districtsSortDir: 'asc',
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
      },
    }
  },
  computed: {
    regions() {
      return this.rawRegions.slice().sort((a, b) => {
        let modifier = 1
        if (this.regionsSortDir === 'desc') modifier = -1
        if (a[this.regionsCurrentSort] < b[this.regionsCurrentSort]) return -1 * modifier
        if (a[this.regionsCurrentSort] > b[this.regionsCurrentSort]) return 1 * modifier
        return 0
      })
    },
    districts() {
      return this.rawDistricts.slice().sort((a, b) => {
        let modifier = 1
        if (this.districtsSortDir === 'desc') modifier = -1
        if (a[this.districtsCurrentSort] < b[this.districtsCurrentSort]) return -1 * modifier
        if (a[this.districtsCurrentSort] > b[this.districtsCurrentSort]) return 1 * modifier
        return 0
      })
    },
  },
  created() {
    this.rawRegions = this.slovakia.regionsData
    this.rawDistricts = this.slovakia.districts
  },
  methods: {
    sortRegions(s) {
      if (s === this.regionsCurrentSort) {
        this.regionsSortDir = this.regionsSortDir === 'asc' ? 'desc' : 'asc'
      }
      this.regionsCurrentSort = s
    },
    sortDistricts(s) {
      if (s === this.districtsCurrentSort) {
        this.districtsSortDir = this.districtsSortDir === 'asc' ? 'desc' : 'asc'
      }
      this.districtsCurrentSort = s
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  width: 100%;
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;
  flex-flow: column wrap;

  .regions {
    min-width: 50%;
    color: blue;
  }
  .district {
    min-width: 50%;
    color: red;
  }
  .lineChart {
    min-width: 50%;
    min-height: 500px;
  }
}
</style>
