<template>
  <div class="app-container claim">
    <div class="violations-process-box st-box">
      <div class="left-box" :class="{ nopadding: showTraffic }">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span class="title-card">Thông tin lưu lượng</span>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterType == 'year' }"
              style="float: right; margin-left: 10px"
              type="info"
              @click="handleObjectType('trafficFlowType', 'year', 'object')"
            >Năm
            </el-button>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterType == 'month' }"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficFlowType', 'month', 'object')"
            >Tháng
            </el-button>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterType == 'day' }"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficFlowType', 'day', 'object')"
            >Ngày
            </el-button>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterType == 'hour' }"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficFlowType', 'hour', 'object')"
            >Giờ
            </el-button>
          </div>
          <div class="body-box-content">
            <div v-if="!showTraffic" class="content-left">
              <div v-if="trafficFlowMetricsByObject" class="infor-traffic-image">
                <div class="text item">
                  <div class="title-label" style="margin-top: 28px ; font-size: 14px;">Tổng số</div>
                  <p class="traffic-number">{{ totalTrafficByObject }}</p>
                </div>
                <div
                  v-for="(val, ind) in trafficFlowMetricsByObject"
                  :key="ind"
                  class="text item"
                >
                  <div class="title-label" style="margin-top: 28px ; font-size: 14px;">{{ val.name }}</div>
                  <p class="traffic-number">{{ val.total }}</p>
                </div>
              </div>
            </div>
          </div>
        </el-card>
      </div>
      <div class="right-box">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span class="title-card">Thông tin sự kiện</span>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterTypeEvent == 'year' }"
              style="float: right; margin-left: 10px"
              type="info"
              @click="handleObjectType('trafficFlowType', 'year', 'status')"
            >Năm
            </el-button>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterTypeEvent == 'month' }"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficFlowType', 'month', 'status')"
            >Tháng
            </el-button>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterTypeEvent == 'day' }"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficFlowType', 'day', 'status')"
            >Ngày
            </el-button>
            <el-button
              class="btn"
              :class="{ 'btn-primary': trafficFilterTypeEvent == 'hour' }"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficFlowType', 'hour', 'status')"
            >Giờ
            </el-button>
          </div>
          <div class="body-box-content">
            <template>
              <div v-if="!showTraffic" class="content-left">
                <div v-if="trafficFlowMetricsByStatus" class="infor-traffic-image">
                  <div class="text item">
                    <div class="title-label" style="margin-top: 28px ; font-size: 14px;">Tổng số</div>
                    <p class="traffic-number">{{ totalTrafficByStatus }}</p>
                  </div>
                  <div
                    v-for="(val, ind) in trafficFlowMetricsByStatus"
                    :key="ind"
                    class="text item item-info-event"
                  >
                    <div class="title-label" style="margin-top: 28px ; font-size: 14px;">{{ val.name }}</div>
                    <p class="traffic-number">{{ val.total }}</p>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </el-card>
      </div>
    </div>

    <div class="violations-process-box nopadding">
      <div class="left-box">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span class="title-card">Biểu đồ lưu lượng</span>
            <el-button
              :class="{ 'btn-primary': trafficByChartFilterType === 'year' }"
              class="btn"
              style="float: right; margin-left: 10px"
              type="info"
              @click="handleObjectType('trafficByChartType', 'year', 'object')"
            >Năm
            </el-button>
            <el-button
              :class="{ 'btn-primary': trafficByChartFilterType === 'month' }"
              class="btn"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficByChartType', 'month', 'object')"
            >Tháng
            </el-button>
            <el-button
              :class="{ 'btn-primary': trafficByChartFilterType === 'day' }"
              class="btn"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficByChartType', 'day', 'object')"
            >Ngày
            </el-button>
          </div>
          <div
            v-if="dataBarChartObject != null"
            class="infor-traffic-image padding-o"
            style="overflow: auto"
          >
            <BarChart :key="flagKeyChartTraffic" :data-object="dataBarChartObject" />
          </div>
        </el-card>
      </div>
      <div class="right-box">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span class="title-card">Biểu đồ sự kiện</span>
            <el-button
              :class="{ 'btn-primary': trafficByChartFilterTypeEvent === 'year' }"
              class="btn"
              style="float: right; margin-left: 10px"
              type="info"
              @click="handleObjectType('trafficByChartType', 'year', 'status')"
            >Năm
            </el-button>
            <el-button
              :class="{ 'btn-primary': trafficByChartFilterTypeEvent === 'month' }"
              class="btn"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficByChartType', 'month', 'status')"
            >Tháng
            </el-button>
            <el-button
              :class="{ 'btn-primary': trafficByChartFilterTypeEvent === 'day' }"
              class="btn"
              style="float: right"
              type="info"
              @click="handleObjectType('trafficByChartType', 'day', 'status')"
            >Ngày
            </el-button>
          </div>
          <div
            v-if="dataBarChartStatus != null"
            class="infor-violations padding-o"
            style="overflow: auto"
          >
            <bar-chart-status :key="flagKeyChartTrafficStatus" :data-object="dataBarChartStatus" />
          </div>
        </el-card>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Cookies from 'js-cookie'
import BarChart from './components/BarChart'
import BarChartStatus from './components/BarChartStatus'

export default {
  name: 'Traffic',
  components: { BarChart, BarChartStatus },
  data() {
    return {
      trafficFilterType: 'day',
      trafficFilterTypeEvent: 'day',
      showTraffic: false,
	    trafficFlowByObject: null,
	    trafficFlowMetricsByObject: null,
	    trafficFlowMetricsByStatus: null,
      intervalId: null,
      totalTrafficByObject: 0,
      totalTrafficByStatus: 0,
	    trafficByChartFilterType: 'day',
	    trafficByChartFilterTypeEvent: 'day',
      trafficFlowByChartMetrics: null,
      flagKeyChartTraffic: 1,
      flagKeyChartTrafficStatus: 1,
      trafficChartData: null,
      trafficFlowChart: null,
	    trafficChartByObject: null,
	    trafficChartByStatus: null,
      dataBarChartObject: [],
      dataBarChartStatus: []
    }
  },
  computed: {},
  created() {
    this.init()
  },
  methods: {
    init() {
      this.getTrafficFlowReport('object', 'day')
      this.getTrafficFlowReport('status', 'day')
      this.getTrafficFlowReportByChart('object', 'day')
      this.getTrafficFlowReportByChart('status', 'day')
      const self = this
	    const idIterval = setInterval(function() {
        switch (self.trafficFilterType) {
          case 'hour':
            self.getTrafficFlowReport('object', 'hour')
            break
          case 'month':
            self.getTrafficFlowReport('object', 'month')
            break
          case 'year':
            self.getTrafficFlowReport('object', 'year')
            break
          default:
            self.getTrafficFlowReport('object', 'day')
            break
        }
        switch (self.trafficFilterTypeEvent) {
          case 'hour':
            self.getTrafficFlowReport('status', 'hour')
            break
          case 'month':
            self.getTrafficFlowReport('status', 'month')
            break
          case 'year':
            self.getTrafficFlowReport('status', 'year')
            break
          default:
            self.getTrafficFlowReport('status', 'day')
            break
        }
        switch (self.trafficByChartFilterType) {
          case 'hour':
            self.getTrafficFlowReportByChart('object', 'hour')
            break
          case 'month':
            self.getTrafficFlowReportByChart('object', 'month')
            break
          case 'year':
            self.getTrafficFlowReportByChart('object', 'year')
            break
          default:
            self.getTrafficFlowReportByChart('object', 'day')
            break
        }
        switch (self.trafficByChartFilterTypeEvent) {
          case 'hour':
            self.getTrafficFlowReportByChart('status', 'hour')
            break
          case 'month':
            self.getTrafficFlowReportByChart('status', 'month')
            break
          case 'year':
            self.getTrafficFlowReportByChart('status', 'year')
            break
          default:
            self.getTrafficFlowReportByChart('status', 'day')
            break
        }
	  }, 30000)
	  this.intervalId = idIterval
      window.localStorage.setItem('intervalId', idIterval)
    },
    handleObjectType(type, value, key) {
      if (type === 'trafficByChartType') {
        if (this.trafficByChartFilterType !== value || this.trafficByChartFilterTypeEvent !== value) {
          if (key == 'object') {
            this.trafficByChartFilterType = value
            this.getTrafficFlowReportByChart(key, value)
          } else {
            this.trafficByChartFilterTypeEvent = value
            this.getTrafficFlowReportByChart(key, value)
          }
        }
      }
      if (type === 'trafficFlowType') {
        if (this.trafficFilterType !== value || this.trafficFilterTypeEvent !== value) {
          if (key == 'object') {
            this.trafficFilterType = value
            this.getTrafficFlowReport(key, value)
          } else {
            this.trafficFilterTypeEvent = value
            this.getTrafficFlowReport(key, value)
          }
        }
      }
    },

    async getTrafficFlowReportByChart(type, timeLevel) {
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      const query = {
        filterBy: type,
        filterTimeLevel: timeLevel
      }
      await axios
        .get(process.env.VUE_APP_API + 'report/chart', {
          headers: headers,
          params: query
        })
        .then((res) => {
          if (res.data) {
            this.trafficChartData = res.data.data
          }
        })
        .catch((err) => {
          console.log(err)
          this.loading = false
          this.$message({
            message: err.response.data.message,
            type: 'error'
          })
        })
      if (this.trafficChartData && type === 'object') {
        this.trafficChartByObject = this.trafficChartData
        this.trafficByChartFilterType = timeLevel
        this.dataBarChartObject = []
        for (const i in this.trafficChartByObject) {
          this.dataBarChartObject.push(this.trafficChartByObject[i])
        }
        this.flagKeyChartTraffic = Math.floor(Math.random() * 1000000)
      }
	    if (this.trafficChartData && type === 'status') {
        this.trafficChartByStatus = this.trafficChartData
        this.trafficByChartFilterTypeEvent = timeLevel
        this.dataBarChartStatus = []
        for (const i in this.trafficChartByStatus) {
          this.dataBarChartStatus.push(this.trafficChartByStatus[i])
        }
        this.flagKeyChartTrafficStatus = Math.floor(Math.random() * 1000000)
      }
    },

    async getTrafficFlowReport(type, timeLevel) {
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      const query = {
        filterBy: type,
        filterTimeLevel: timeLevel
      }
      await axios
        .get(process.env.VUE_APP_API + 'report', {
          headers: headers,
          params: query
        })
        .then((res) => {
          if (res.data) {
            this.trafficFlowByObject = res.data.data
          }
        })
        .catch((err) => {
          console.log(err)
          this.loading = false
          this.$message({
            message: err.response.data.message,
            type: 'error'
          })
        })
      if (this.trafficFlowByObject && type === 'object') {
        this.trafficFlowMetricsByObject = this.trafficFlowByObject
        this.dataPieTraffic = this.handleGetValuePieChartInforTraffic()
        this.trafficFilterType = timeLevel

        for (const i in this.trafficFlowMetricsByObject) {
          if (this.trafficFlowMetricsByObject[i].code === 0) {
            this.trafficFlowMetricsByObject.splice(i, 1)
            break
          }
        }
      }
	  if (this.trafficFlowByObject && type === 'status') {
        this.trafficFlowMetricsByStatus = this.trafficFlowByObject
        this.dataPieTraffic = this.handleGetValuePieChartInforTrafficByStatus()
        this.trafficFilterTypeEvent = timeLevel

        for (const i in this.trafficFlowMetricsByStatus) {
          if (this.trafficFlowMetricsByStatus[i].code === 0) {
            this.trafficFlowMetricsByStatus.splice(i, 1)
            break
          }
        }
      }
    },

    handleGetValuePieChartInforTraffic() {
      if (
        this.trafficFlowMetricsByObject
      ) {
        this.totalTrafficByObject = 0
      }
      this.trafficFlowMetricsByObject.forEach((element) => {
        this.totalTrafficByObject = this.totalTrafficByObject + element.total
      })
      this.totalTrafficByObject = this.formatNumber(this.totalTrafficByObject)
      var t = this.trafficFlowMetricsByObject.map((m) => {
        return { name: m.name, y: m.total }
      })
      return t
    },

    handleGetValuePieChartInforTrafficByStatus() {
      if (
        this.trafficFlowMetricsByStatus
      ) {
        this.totalTrafficByStatus = 0
      }
      this.trafficFlowMetricsByStatus.forEach((element) => {
        this.totalTrafficByStatus = this.totalTrafficByStatus + element.total
      })
      this.totalTrafficByStatus = this.formatNumber(this.totalTrafficByStatus)
      var t = this.trafficFlowMetricsByStatus.map((m) => {
        return { name: m.name, y: m.total }
      })
      return t
    },
    formatNumber(amount, decimalCount = 2, decimal = '.', thousands = ',') {
      try {
        decimalCount = Math.abs(decimalCount)
        decimalCount = isNaN(decimalCount) ? 2 : decimalCount

        const negativeSign = amount < 0 ? '-' : ''

        const i = parseInt(
          (amount = Math.abs(Number(amount) || 0).toFixed(decimalCount))
        ).toString()
        const j = i.length > 3 ? i.length % 3 : 0

        return (
          negativeSign +
            (j ? i.substr(0, j) + thousands : '') +
            i.substr(j).replace(/(\d{3})(?=\d)/g, '$1' + thousands)
        )
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.el-button + .el-button {
  margin-left: 10px;
}

.no-margin-top {
  margin-top: 0;
}

.block {
  display: inline-block;
  margin-right: 20px;
}

.list-detail .item {
  margin-bottom: 10px;

  .label {
    display: inline-block;
    width: 100px;
  }

  img {
    max-width: 100%;
    display: inline-block;
  }
}

.box-card .el-card__header button {
    padding: 3px 0;
    font-size: 14px;
    height: 27px;
    min-width: 62px;
    border-radius: 14px;
    background-color: #f2f2f2;
    color: rgba(74,74,74,.87);
    border: none;
}

.box-card .el-card__header .btn-primary {
    color: hsla(0,0%,100%,.87);
    background-color: #292663;
}

.box-card .title-card {
    font-size: 16px;
    font-weight: 700;
}
</style>

<style lang="scss">
.item-info-event:nth-child(3) {
	min-width: 115px;
}

.infor-violations-table {
  tbody tr td:first-child .cell {
    /*padding-left: 0 !important;*/
  }
}

.form-value {
  .el-input__inner {
    margin-bottom: 10px;
  }
}

.claim td .el-button {
  margin-left: 0;
}

.st-box {
  .el-card__body {
    height: calc(100% - 64.22px);
  }
}

.nopadding {
  .el-card__body {
    padding: 0;
  }
}

.aleart-infor {
  .el-card__body {
    padding-right: 0px;
  }
}

.el-table tr:nth-child(odd) {
  background-color: #F8F8F8;
}

.table__scroll {
  .el-card__body {
    overflow: auto;
    padding-top: 15px;
    max-height: 405px;

    th div {
      text-align: left;
    }
  }

  .right-box {
    th div {
      text-align: left;
      padding-left: 2px;
    }
  }
}

.violations-process-box {
  display: flex;
  font-size: 14px;
  width: 100%;
  margin-bottom: 17px;

  .arrow-dashboard {
    position: relative;
    display: inline-block;
    top: 2px;

    .arrow-left {
      cursor: pointer;
      width: 12px !important;
      height: 12px !important;
      display: inline-block;
      border: 8px solid;
      border-color: transparent #9898984a transparent transparent;

      &.active {
        border-color: transparent #989898 transparent transparent;
      }

      &:active {
        transform: scale(1.1);
      }
    }

    .arrow-right {
      cursor: pointer;

      &:active {
        transform: scale(1.1);
      }

      margin-left: 3px;
      width: 12px !important;
      display: inline-block;
      height: 12px !important;
      border: 8px solid;
      border-color: transparent transparent transparent #9898984a;

      &.active {
        border-color: transparent transparent transparent #989898;
      }
    }
  }

  .infor-traffic-alert {
    color: rgba(0, 0, 0, 0.6);
    overflow: auto;
    max-height: 500px;

    tr {
      height: 50px;
    }

    .site-name {
      .site-name-box {
        display: flex;
        align-items: center;
      }

      img {
        width: 13.4px;
        height: 12px;
        margin-right: 5px;
      }

      min-width: 120px;
      padding-right: 5px;
    }

    .statusTrafficName {
      padding-left: 10px;
      width: 100% !important;
    }

    .currentTime {
      min-width: 140px;
      margin-left: auto;
    }
  }

  .alert-system {
    overflow: auto;
    color: rgba(0, 0, 0, 0.6);

    tr {
      height: 50px;
    }

    td {
      width: 33%;
    }

    .image-box {
      .site-name-box {
        display: flex;
        align-items: center;
      }

      img {
        width: 13.4px;
        height: 12px;
        margin-right: 5px;
      }

      min-width: 100px;
      padding-right: 5px;
    }

    .currentTime {
      min-width: 140px;
      margin-left: auto;
    }
  }

  .right-box,
  .left-box {
    width: calc((100% - 18px) / 2);

    .box-card {
      .infor-violations {
        width: 100%;

        .infor-violation-top,
        .infor-violation-bottom {
          display: flex;
        }

        .box-item {
          display: flex;
          flex-direction: column;
          align-items: center;
          width: 33.33%;
          position: relative;
          height: 100%;
          color: #fff;
          background-color: red;

          .box-item-title {
            font-size: 14px;

            margin-top: 15px;
          }

          .box-item-number {
            font-size: 22px;
            top: 50%;
            position: absolute;
            transform: translateY(-50%);
          }
        }

        justify-content: space-around;

        .circle-violation-number {
          display: flex;
          align-items: center;
          align-content: center;
          justify-content: center;

          &.totalViolation {
            border-color: #f9327a;
            color: #f9327a;
          }

          &.totalViolationConfirm {
            border-color: #35bae9;
            color: #35bae9;
          }

          &.totalViolationProcess {
            border-color: #fcc103;
            color: #fcc103;
          }

          width: 100px;
          height: 100px;
          border-radius: 50%;
          border: 5px solid #35bae9;
          position: relative;

          .number-circle-violation {
            font-size: 45px;
          }
        }

        .type-info {
          margin: 0px;
          text-align: center;
          height: 35px;
        }
      }

      .infor-traffic-image {
        display: flex;
        height: 100%;
        // min-height: 141px;
        .item {
          text-align: center;
          width: 100%;
          justify-content: center;
          align-items: center;
          display: flex;
          flex-direction: column;

          img {
            margin-top: 14px;

            &.constructed {
              height: 38px;
              margin-top: 10px;
            }

            &.bike {
              height: 42px;
              margin-top: 10px;
            }

            &.pedestrian {
              height: 40px;
              margin-top: 10px;
            }
          }

          p {
            color: #505050;
            font-size: 16px;
            margin: 28px 19px;
            font-weight: bold;
          }
        }
      }
    }
  }

  .icon-th {
    width: 30px;
    max-height: 30px;
  }

  .body-box-content {
    height: 100%;

    .content-left {
      height: 100%;
    }
  }

  .right-box-content {
    display: flex;
    height: 100%;

    .infor-violations-box {
      .infor-violation-top,
      .infor-violation-bottom {
        width: 100%;
        height: 100%;

        .box-item {
          height: 100%;
        }
      }
    }

    .infor-traffic-pie-chart {
      width: 100%;
    }
  }

  .svg-icon {
    /* width: 1em; */
    height: 30px;
    width: 30px;
  }

  .icon-th-pedestrian {
    width: 20px;
    height: 22px;
  }

  .right-box {
    margin-left: 9px;
  }

  .left-box {
    margin-right: 9px;
  }

  .setpadding {
    .el-card__body {
      padding: 0;
      height: calc(100% - 64.22px);
    }
  }

  svg.highcharts-root {
    font-family: 'Roboto-Medium', 'Roboto', Helvetica Neue, Helvetica, PingFang SC, Hiragino Sans GB, Microsoft YaHei, Arial, sans-serif !important;
  }

  .highcharts-container {
    span {
      font-family: 'Roboto-Medium', 'Roboto', Helvetica Neue, Helvetica, PingFang SC, Hiragino Sans GB, Microsoft YaHei, Arial, sans-serif !important;
    }
  }
}
</style>
