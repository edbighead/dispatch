<template>
  <v-card :loading="loading">
    <v-card-title>Mean Days Active (Active -> Stable)</v-card-title>
    <apexchart type="line" height="250" :options="chartOptions" :series="series"></apexchart>
  </v-card>
</template>

<script>
import _ from "lodash"
import differenceInCalendarDays from "date-fns/differenceInCalendarDays"
import parseISO from "date-fns/parseISO"
import VueApexCharts from "vue-apexcharts"
export default {
  name: "IncidentActiveTimeCard",

  props: {
    value: {
      type: Object,
      default: function() {
        return {}
      }
    },
    interval: {
      type: String,
      default: function() {
        return "Month"
      }
    },
    loading: {
      type: Boolean,
      default: function() {
        return false
      }
    }
  },

  components: {
    apexchart: VueApexCharts
  },

  data() {
    return {}
  },

  // TODO convert to reported_at
  computed: {
    series() {
      let series = { name: "Average Days Active", data: [] }
      _.forEach(this.value, function(value, key) {
        series.data.push(
          Math.round(
            _.sumBy(value, function(item) {
              let endTime = new Date().toISOString()
              if (item.stable_at) {
                endTime = item.stable_at
              }
              return differenceInCalendarDays(parseISO(endTime), parseISO(item.created_at))
            }) / value.length
          )
        )
      })

      return [series]
    },
    chartOptions() {
      return {
        chart: {
          height: 350,
          type: "line",
          toolbar: {
            show: false
          }
        },
        xaxis: {
          categories: Object.keys(this.value) || [],
          title: {
            text: this.interval
          }
        },
        dataLabels: {
          enabled: true
        },
        stroke: {
          curve: "smooth"
        },
        markers: {
          size: 1
        },
        yaxis: {
          title: {
            text: "Days"
          }
        },
        legend: {
          position: "top"
        }
      }
    }
  }
}
</script>
