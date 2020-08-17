<!-- Template Tag can not be merged... -->

<script lang="ts">
import { Component, Emit, Prop, Watch, mixins } from 'nuxt-property-decorator'
// import Chart from 'chart.js'
import VueChart from 'vue-chartjs'
// 棒グラフの場合は、Barを使う
// eslint-disable-next-line
const Bar = VueChart.Pie

@Component
// mixinsもBarに変更
export default class ChartLine extends mixins(Bar) {
  @Prop({ default: [] }) chartData!: any

  @Watch('chartData', { deep: true })
  watchTmp() {
    this.main()
  }

  mounted() {
    this.main()
  }

  @Emit('clickChart')
  clickChart(data: any) {
    return data
  }

  main() {
    const chartData = this.chartData

    const self = this
    const chartOption: any = {
      // アスペクト比を固定しないように変更
      maintainAspectRatio: false,
      onClick(evt: any, item: any[]) {
        const label = item[0]._model.label
        self.clickChart({
          property: chartData.property,
          label,
          evt,
        })
      },
    }

    this.renderChart(chartData, chartOption)
  }
}
</script>
