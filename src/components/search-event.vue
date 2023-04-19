<template>
  <span class="search-template">
    <el-popover
      v-model="visible"
      placement="bottom-start"
      title=""
      width="380"
      :visible-arrow="false"
      trigger="click"
    >
      <div class="search-form-content">
        <el-form ref="formSearch" :model="filter" :rules="rules">
          <el-form-item label="Thời gian" prop="date">
            <el-date-picker
              v-model="filter.date"
              type="datetimerange"
              :clearable="false"
              range-separator="-"
              start-placeholder="Bắt đầu"
              format="yyyy-MM-dd HH:mm:ss"
              end-placeholder="Kết thúc"
            />
          </el-form-item>
          <el-form-item label="Vị trí">
            <el-select
              v-model="filter.sites"
              multiple
              filterable
              clearable
              collapse-tags
              :class="{ multi: filter.sites && filter.sites.length > 1 }"
              class="site-search-popup"
              placeholder="Chọn vị trí"
            >
              <el-option
                v-for="item in siteLst"
                :key="item.siteId"
                :label="item.siteName"
                :value="item.siteId"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="Nguồn phát hiện">
            <el-select v-model="filter.sourceType" placeholder="Lựa chọn">
              <el-option
                v-for="item in sourceObjectList"
                :key="item.val"
                :label="item.name"
                :value="item.val"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="Thiết bị phát hiện">
            <el-select
              v-model="filter.cameras"
              multiple
              filterable
              clearable
              collapse-tags
              :class="{ multi: filter.cameras && filter.cameras.length > 1 }"
              class="site-search-popup"
              placeholder="Chọn nguồn"
            >
              <el-option
                v-for="item in cameraLst"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-row :gutter="20">
            <el-col
              :span="24"
            ><div class="grid-content bg-purple">
              <el-form-item label="Loại sự kiện">
                <el-select
                  v-model="filter.violationTypes"
                  clearable
                  multiple
                  filterable
                  collapse-tags
                  :class="{ multi: filter.violationTypes && filter.violationTypes.length > 1 }"
                  placeholder="Tất cả"
                >
                  <el-option
                    v-for="tmpType in violationTypeLst"
                    :key="tmpType.id"
                    :label="tmpType.name"
                    :value="tmpType.code"
                  />
                </el-select>
              </el-form-item></div></el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <div class="grid-content bg-purple">
                <el-form-item label="Trạng thái">
                  <el-select
                    v-model="filter.status"
                    clearable
                    placeholder="Chọn trạng thái"
                  >
                    <el-option
                      v-for="tmpType in statusLst"
                      :key="tmpType.code"
                      :label="tmpType.name"
                      :value="tmpType.id"
                    />
                  </el-select>
                </el-form-item></div></el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="24">
              <div class="grid-content bg-purple">
                <el-form-item label="Trạng thái kiểm duyệt">
                  <el-select v-model="filter.reportStatus" placeholder="Lựa chọn">
                    <el-option
                      v-for="item in statusListCheck"
                      :key="item.val"
                      :label="item.name"
                      :value="item.val"
                    />
                  </el-select>
                </el-form-item>
              </div></el-col>
          </el-row>
          <el-form-item class="action">
            <el-button
              type="info"
              @click="refresh()"
            ><i class="el-icon-refresh" /> Làm mới</el-button>
            <el-button type="primary" @click="search()">Áp dụng</el-button>
          </el-form-item>
        </el-form>
      </div>
      <span slot="reference">
        <img :src="filter_icon" alt="">
      </span>
    </el-popover>
  </span>
</template>
<script type="text/javascript">
import filter_icon from '@/assets/images/filter.png'
import moment from 'moment'
import Cookies from 'js-cookie'
import axios from 'axios'

export default {
  props: {
    searchEvent: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      filter_icon: filter_icon,
      visible: false,
      loading: false,
      eventHandMade: false,
      siteLst: [],
      cameraLst: [],
      vehicleTypeLst: [],
      statusLst: [],
      statusListCheck: [
        {
          val: '',
          name: 'Tất cả'
        },
        {
          val: false,
          name: 'Chưa kiểm duyệt'
        },
        {
          val: true,
          name: 'Đã kiểm duyệt'
        }
      ],
      sourceObjectList: [
        {
          val: '',
          name: 'Tất cả'
        },
        {
          val: 'AUTO',
          name: 'Tự động'
        },
        {
          val: 'MANUAL',
          name: 'Thủ công'
        }
      ],
      statusListEvent: {
        NOT_SEEN: 1,
        VERIFICATION: 2,
        PROCESSING: 3,
        PROCESSED: 4,
        INCORRECT: 5
      },
      groupsList: [],
      violationTypeLst: [],
      exceptViolation: [],
      objectTypeLst: [],
      filter: {
        status: null,
        manual: null,
        violationTypes: null,
        objectType: null,
        date: [
          moment().subtract(30, 'day').startOf('day').format('YYYY-MM-DD HH:mm:ss'),
          moment().endOf('day').format('YYYY-MM-DD HH:mm:ss')
        ],
        sites: [],
        cameras: [],
        group: null,
        reportStatus: '',
        sourceType: ''
      },
      rules: {
        date: [
          {
            required: true,
            message: 'Khoảng thời gian tìm kiếm là bắt buộc',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.init()
  },
  mounted() {
    this.handleRedirectNotify()
  },
  methods: {
    async init() {
      console.log('this.search', this.searchEvent)
      if (this.searchEvent && this.searchEvent.fromDate && this.searchEvent.toDate) {
        this.filter.date = [this.searchEvent.fromDate, this.searchEvent.toDate]
      }
      if (this.searchEvent && this.searchEvent.site) {
        this.filter.sites = [this.searchEvent.site]
      }
      if (this.searchEvent && this.searchEvent.eventTypeList) {
        this.filter.violationTypes = this.searchEvent.eventTypeList
      }
      // this.getSiteList()
      // this.getCameraList()
      // this.getViolationTypeList()
    },
    handleRedirectNotify() {
      let shiftInfo = localStorage.getItem('shiftNotify')
      if (shiftInfo) {
        shiftInfo = JSON.parse(shiftInfo)
        this.filter.date = [shiftInfo.startTime, shiftInfo.endTime]
        this.filter.status = this.statusListEvent[shiftInfo.status]
        localStorage.removeItem('shiftNotify')
        this.search()
      }
    },
    // async getSiteList() {
    //   const queryCustom = {
    //     page: 0,
    //     size: 2000
    //   }
    //   const result = await siteResource.list(queryCustom)
    //   if (result && result.data) {
    //     this.siteLst = result.data
    //   }
    // },
    // async getCameraList() {
    //   const siteIds = []
    //   this.filter.sites.map((site) => {
    //     siteIds.push(site)
    //   })
    //   const queryCustom = {
    //     page: 0,
    //     size: 10000
    //   }
    //   const result = await cameraResource.list(queryCustom)
    //   if (result && result.data) {
    //     this.cameraLst = result.data
    //   } else {
    //     this.cameraLst = []
    //   }
    //   this.filter.cameras = []
    // },
    // async getObjectTypeList() {
    //   const queryCustom = {
    //     catType: 'OBJECT_TYPE_GSAN'
    //   }
    //   const result = await categoryResource.list(queryCustom)
    //   if (result && result.data) {
    //     this.objectTypeLst = result.data
    //     this.objectTypeLst.unshift({ code: null, id: '', name: 'Tất cả' })
    //   }
    // },
    // async statusList() {
    //   const result = await statusResource.list()
    //   if (result && result.data) {
    //     this.statusLst = result.data
    //     this.statusLst.unshift({ value: 'Tất cả', name: null })
    //   }
    // },
    // async getGroupList() {
    //   const queryCustom = {
    //     page: 0,
    //     size: 1000
    //   }
    //   const { data } = await groupsResource.list(queryCustom)
    //   if (data) {
    //     this.groupsList = data
    //     this.groupsList.unshift({ uuid: null, name: 'Tất cả' })
    //   } else {
    //     this.groupsList = []
    //   }
    // },
    // async getViolationTypeList() {
    //   const queryCustom = {
    //     catType: 'EVENT'
    //   }
    //   const self = this
    //   const result = await categoryResource.list(queryCustom)
    //   if (result && result.data) {
    //     this.violationTypeLst = result.data.filter(function(val) {
    //       return !self.exceptViolation.includes(val.code)
    //     })
    //     // this.violationTypeLst.unshift({ id: '', code: null, name: 'Tất cả' })
    //   }
    // },
    search() {
      this.$refs.formSearch.validate((valid) => {
        if (valid) {
          const queryCustom = {
            startDate: '',
            endDate: '',
            site: null,
            manual: null,
            reportStatus: this.filter.reportStatus ? true : this.filter.reportStatus === false ? false : '',
            objectName: this.eventHandMade ? 'manual' : 'all',
            camera: null,
            status: this.filter.status ? this.filter.status : '',
            // violationType: this.filter.violationType ? this.filter.violationType : '',
            objectType: this.filter.objectType ? this.filter.objectType : '',
            sourceType: this.filter.sourceType
          }
          if (this.filter.date) {
            const startDate = moment(this.filter.date[0]).format(
              'YYYY-MM-DD HH:mm:ss'
            )
            const endDate = moment(this.filter.date[1]).format(
              'YYYY-MM-DD HH:mm:ss'
            )
            queryCustom.startDate = startDate
            queryCustom.endDate = endDate
          }
          if (this.filter.sites && this.filter.sites.length > 0) {
            queryCustom.site = this.filter.sites.toString()
          } else {
            queryCustom.site = ''
          }
          if (this.filter.cameras && this.filter.cameras.length > 0 && !this.eventHandMade) {
            queryCustom.camera = this.filter.cameras.toString()
          } else {
            queryCustom.camera = ''
          }
          if (this.filter.violationTypes && this.filter.violationTypes.length > 0) {
            queryCustom.violationType = this.filter.violationTypes.toString()
          } else {
            queryCustom.violationType = ''
          }
          if (this.eventHandMade) {
            queryCustom.camera = ''
            queryCustom.manual = this.filter.manual
          }
          this.visible = false
          this.$emit('clicked', queryCustom)
        } else {
          return false
        }
      })
    },
    refresh() {
      this.eventHandMade = false
      this.filter = {
        plate: null,
        vehicleType: null,
        violationTypes: null,
        date: [
          moment().subtract(2, 'day').startOf('day').format('YYYY-MM-DD HH:mm:ss'),
          moment().endOf('day').format('YYYY-MM-DD HH:mm:ss')
        ],
        sites: [],
        cameras: [],
        group: null,
        status: null,
        sourceType: ''
      }
    }
  }
}
</script>
  <style type="text/css" lang="scss">
  .search-template {
    .el-popover__reference {
      background: initial;
      border: initial;
      cursor: pointer;
      margin-left: 12px;
      position: relative;
      top: 8px;
    }
  }

  .search-form-content {
    .el-form-item__label {
      width: 100%;
      text-align: left;
    }

    .el-date-editor,
    .el-select {
      width: 100%;
    }

    .el-form-item {
      margin-bottom: 8px;
    }

    .action {
      margin-top: 10px;
      margin-bottom: 20px;

      .el-button--info {
        border-radius: 4px;
        background: #26c6da;
        color: white;
        border: 1px solid #26c6da;
        font-size: 16px;
        font-weight: 500;
        width: 147px;
      }

      .el-button--primary {
        border-radius: 4px;
        background: #1f7ff5;
        color: white;
        border: 1px solid #1f7ff5;
        font-size: 16px;
        font-weight: 500;
        width: 147px;
        float: right;
      }
    }
  }
  </style>

