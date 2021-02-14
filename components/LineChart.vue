<script>
import { Line } from 'vue-chartjs'

export default {
  extends: Line,
  props: {
    label: {
      type: String,
      required: true,
    },
    chartData: {
      type: Array,
      required: true,
    },
    options: {
      type: Object,
      required: true,
    },
  },
  mounted() {
    const dates = this.chartData
      .slice()
      .map((item) => item.date)
      .reverse()
    const cases = this.chartData
      .slice()
      .map((item) => item.cases)
      .reverse()
    this.renderChart(
      {
        labels: dates,
        datasets: [
          {
            label: this.label,
            data: cases,
          },
        ],
      },
      this.options
    )
  },
}
</script>
