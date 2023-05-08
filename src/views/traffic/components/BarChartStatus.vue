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

const animationDuration = 500

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
          },
          textStyle: {
            fontFamily: 'Arial',
            fontSize: 14
          },
          formatter: function(params, ticket, callback) {
            const label = params[0].axisValue
            const total = params.reduce((res, item) => {
              res += item.value
              return res
            }, 0)

            const lastText = params.reduce((res, item) => {
              res += '<div style="display:flex; justify-content: flex-start; align-item: center; margin-top: 10px">'
              res += '<div style="width:10px; height:10px; margin-right:5px; border-radius: 50%; background-color:' + item.color + '"></div>'
              res += '<span style="line-height:14px">' + item.seriesName + ': ' + item.value + '</span>'
              res += '</div>'
              return res
            }, '')
            return label + '<br/> Tổng: ' + total + '<br/>' + lastText
          }
          // formatter: '{a0}: {b0} - {c0}<br/> {a1}: {b1} - {c1}<br/> {a2}: {b2} - {c2}<br/> {a3}: {b3} - {c3}<br/> {a4}: {b4} - {c4}'
        },
        xAxis: [{
          type: 'category',
          data: this.handleChartKey(),
          axisTick: {
            alignWithLabel: true
          }
        }],
        grid: {
          width: '92%',
          left: '6%',
          top: 10 // Khoảng cách với trên cùng
        },
        legend: {
          data: ['Chưa xem', 'Xác minh sự kiện', 'Đang xử lý', 'Đã xử lý', 'Báo sai'],
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
          name: 'Chưa xem',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(1),
          animationDuration
        }, {
          name: 'Xác minh sự kiện',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(2),
          animationDuration
        }, {
          name: 'Đang xử lý',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(3),
          animationDuration
        }, {
          name: 'Đã xử lý',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(4),
          animationDuration
        }, {
          name: 'Báo sai',
          type: 'bar',
          stack: 'vistors',
          barWidth: '60%',
          data: this.handleGetValueWithKey(5),
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
