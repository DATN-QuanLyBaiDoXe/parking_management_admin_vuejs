<template>
  <div
    :class="className"
    :style="{height:height,width:width}"
    style="padding-top: 10px;"
  />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from '@/mixins/resize'

const animationDuration = 3000

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '450px'
    },
    dataObject: {
      type: Array,
      default: null
    }
  },
  data() {
    return {
      chart: null,
      dataBarChart: [],
	  label: []
    }
  },
  mounted() {
    this.data = this.dataObject
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')

      this.chart.setOption({
        tooltip: {
          trigger: 'axis',
          axisPointer: { // 坐标轴指示器，坐标轴触发有效
            type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
          }
        },
        xAxis: [{
          type: 'category',
          data: this.handleChartKey(),
          axisTick: {
            alignWithLabel: true
          }
        }],
        grid: {
          top: 10 // Khoảng cách với trên cùng
        },
        legend: {
          data: ['Ô tô', 'Xe máy', 'Xe đạp điện', 'Xe đạp'],
          left: 'center',
          bottom: 6,
		  textStyle: {
            fontFamily: 'Arial',
            fontSize: 14
          }
        },
        yAxis: [{
          type: 'value',
          axisTick: {
            show: false
          }
        }],
        series: [{
          name: 'Ô tô',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(1),
          animationDuration
        }, {
          name: 'Xe máy',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(2),
          animationDuration
        }, {
          name: 'Xe đạp điện',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(3),
          animationDuration
        }, {
          name: 'Xe đạp',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(4),
          animationDuration
        }, {
          name: 'Tổng cộng',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetLine(),
          animationDuration
        }]
      })
    },

    handleChartKey() {
      return this.dataObject.map((a) => a.time)
    },
    handleGetLine() {
      return this.dataObject.map((m) => {
        let total = 0
        if (m.reportDTOList !== null) {
          for (let i = 0; i < m.reportDTOList.length; i++) {
            total += m.reportDTOList[i].total
          }
        }
        return total
      })
    },
    handleGetValueWithKey(key) {
      return this.dataObject.map((m) => {
        if (m.reportDTOList !== null) {
          const codeArr = m.reportDTOList.map((n) => { return n.code })
		  if (!codeArr.includes(key)) {
            return 0
		  }
          for (let i = 0; i < m.reportDTOList.length; i++) {
            if (m.reportDTOList[i].code == key) {
              return m.reportDTOList[i].total
            }
          }
        } else {
          return 0
        }
      })
    }
  }
}
</script>
