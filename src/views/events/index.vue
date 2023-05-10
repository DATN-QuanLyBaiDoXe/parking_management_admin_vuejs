<!-- eslint-disable vue/valid-v-model -->
<template>
  <div class="body">
    <div>
      <!-- <el-input
        v-model="queryPage.search"
        class="input-search"
        placeholder="Tìm kiếm"
        prefix-icon="el-icon-search"
        @change="onChangeInputSearch"
      /> -->
      <el-input
        v-model="queryPage.search"
        class="input-search"
        placeholder="Tìm kiếm"
        prefix-icon="el-icon-search"
        @keyup.enter.native="handleFilter()"
        @change="onChangeInputSearch"
      />
      <Search
        :key="flagKeySearch"
        :search-event="searchData"
        @clicked="onClickChild"
      />
      <el-button
        class="btn-add"
        type="primary"
        icon="el-icon-plus"
        @click="handleCreate()"
      >Thêm mới</el-button>
    </div>
    <div class="mb-4">
      <div id="vehicle-table">
        <!-- <div class="table-responsive"> -->
        <div v-if="multiSelected.length > 0" id="select-all-delete">
          <i class="el-icon-close" @click="closeDelete()" />
          <span
            v-if="multiSelected.length < 10"
          >0{{ multiSelected.length }}</span><span v-else>{{ multipleSelection.length }}</span> mục được chọn
          <el-button
            type="danger"
            :loading="loading_delete_all"
            @click="destroyMulti()"
          ><i class="el-icon-delete" /> Xóa</el-button>
        </div>
        <el-table
          ref="multipleTable"
          v-loading="loading"
          :data="eventList"
          style="width: 100%"
          @selection-change="handleSelectionChange"
          @row-click="rowClicked"
        >
          <el-table-column type="selection" width="50" />
          <el-table-column label="STT" width="50">
            <template slot-scope="scope">{{
              scope.$index +
                1 +
                (queryPage.page > 0 ? queryPage.page - 1 : 0) * queryPage.size
            }}</template>
          </el-table-column>
          <el-table-column label="Thời gian phát hiện">
            <template slot-scope="scope">{{
              scope.row.createdDate | formatDatetime
            }}</template>
          </el-table-column>
          <el-table-column label="Loại sự kiện">
            <template slot-scope="scope">{{
              scope.row.eventType | getEventType
            }}</template>
          </el-table-column>
          <el-table-column label="Loại phương tiện">
            <template slot-scope="scope">{{
              scope.row.objectType | getObjectType
            }}</template>
          </el-table-column>
          <el-table-column label="Nguồn phát hiện">
            <template slot-scope="scope">{{
              scope.row.sourceType | getSourceType
            }}</template>
          </el-table-column>
          <el-table-column prop="place" label="Biển số" />
          <el-table-column label="Trạng thái">
            <template slot-scope="scope">{{
              scope.row.status | getStatus
            }}</template>
          </el-table-column>
        </el-table>
        <el-pagination
          :current-page.sync="queryPage.page"
          :page-sizes="[1, 5, 10, 15, 20]"
          :page-size="queryPage.size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="queryPage.total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
      <div id="detail-vehicle" class="violation-ruleForm" style="width: 0">
        <i class="el-icon-close" @click="closeDetail()" />
        <template v-if="eventDetail">
          <div class="event-image">
            <img
              v-if="eventDetail.image"
              height="200px"
              width="100%"
              :src="eventDetail.image"
            >
            <img v-else height="200px" width="100%" :src="user_default">
          </div>

          <template v-if="eventDetail.eventType">
            <div class="item-label">Loại sự kiện</div>
            <div class="item-text">{{ eventDetail.eventType | getEventType }}</div>
          </template>

          <template v-if="eventDetail.objectType">
            <div class="item-label">Loại phương tiện</div>
            <div class="item-text">
              {{ eventDetail.objectType | getObjectType }}
            </div>
          </template>

          <template v-if="eventDetail.sourceType">
            <div class="item-label">Nguồn phát hiện</div>
            <div class="item-text">
              {{ eventDetail.sourceType | getSourceType }}
            </div>
          </template>

          <template v-if="eventDetail.place">
            <div class="item-label">Biển số</div>
            <div class="item-text">{{ eventDetail.place }}</div>
          </template>

          <template v-if="eventDetail.status">
            <div class="item-label">Trạng thái</div>
            <div class="item-text">{{ eventDetail.status | getStatus }}</div>
          </template>

          <template v-if="eventDetail.description">
            <div class="item-label">Ghi chú</div>
            <div class="item-text">{{ eventDetail.description }}</div>
          </template>

          <div
            class="action"
            style="display: flex; justify-content: space-between;"
          >
            <el-button
              type="info"
              @click="handleEdit(eventDetail)"
            ><i class="el-icon-edit" /> Sửa
            </el-button>
            <el-button
              type="danger"
              :loading="loadingVehicle"
              @click="destroy(eventDetail)"
            ><i class="el-icon-delete" /> Xóa
            </el-button>
            <el-button
              type="warning"
              :loading="loadingVehicle"
              @click="updateEventStatus(eventDetail.uuid, 5)"
            ><i class="el-icon-warning-outline" /> Báo sai
            </el-button>
          </div>
        </template>
      </div>
    </div>

    <el-dialog
      title="Thêm mới"
      :visible.sync="dialogAdd"
      width="500px"
      :close-on-click-modal="false"
      @close="closeDialog('addForm')"
    >
      <el-form
        ref="addForm"
        :model="eventInfo"
        :rules="rules"
        label-position="top"
        label-width="200px"
      >
        <div class="block-item">
          <div class="item-mid">
            <el-form-item style="margin-bottom: 21px" label="Loại sự kiện" prop="eventType">
              <el-select
                v-model="eventInfo.eventType"
                placeholder="Chọn loại sự kiện"
                style="width: 100%"
              >
                <el-option
                  v-for="type in eventTypeList"
                  :key="type.value"
                  :label="type.label"
                  :value="type.value"
                />
              </el-select>
            </el-form-item>
            <el-form-item style="margin-bottom: 21px" label="Loại phương tiện" prop="objectType">
              <el-select
                v-model="eventInfo.objectType"
                placeholder="Chọn loại phương tiện"
                style="width: 100%"
              >
                <el-option
                  v-for="type in objectTypeList"
                  :key="type.value"
                  :label="type.label"
                  :value="type.value"
                />
              </el-select>
            </el-form-item>
            <el-form-item v-if="[1,2,3].includes(eventInfo.objectType)" style="margin-bottom: 21px" label="Biển số" prop="place">
              <el-input v-model="eventInfo.place" maxlength="9" />
            </el-form-item>
            <el-form-item v-else style="margin-bottom: 21px" label="Biển số">
              <el-input v-model="eventInfo.place" maxlength="9" />
            </el-form-item>
            <!-- <el-form-item style="margin-bottom: 21px" label="Ảnh" prop="image">
              <el-input
                v-model="eventInfo.image"
              />
            </el-form-item> -->
            <el-form-item label="Hình ảnh">
              <UploadImage
                :key="keyResetUpload"
                :list-image="listFile"
                @removeUploadImage="removeUploadImageHandle"
                @getListFile="getListFile"
              />
            </el-form-item>
            <el-form-item style="margin-bottom: 21px" label="Ghi chú" prop="description">
              <el-input v-model="eventInfo.description" type="textarea" rows="3" />
            </el-form-item>
          </div>
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button class="cancel-btn" type="info" @click="dialogAdd = false">Hủy</el-button>
        <el-button
          type="primary"
          :loading="loading_add"
          @click="addEventHandle()"
        >Lưu
        </el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="Cập nhật"
      :visible.sync="dialogEdit"
      width="500px"
      :close-on-click-modal="false"
      @close="closeDialog('editForm')"
    >
      <el-form
        ref="editForm"
        :model="eventInfo"
        :rules="rules"
        label-position="top"
        label-width="200px"
      >
        <div class="block-item">
          <div class="item-mid">
            <el-form-item style="margin-bottom: 21px" label="Loại sự kiện" prop="eventType">
              <el-select
                v-model="eventInfo.eventType"
                placeholder="Chọn loại sự kiện"
                style="width: 100%"
              >
                <el-option
                  v-for="type in eventTypeList"
                  :key="type.value"
                  :label="type.label"
                  :value="type.value"
                />
              </el-select>
            </el-form-item>
            <el-form-item style="margin-bottom: 21px" label="Loại phương tiện" prop="objectType">
              <el-select
                v-model="eventInfo.objectType"
                placeholder="Chọn loại phương tiện"
                style="width: 100%"
              >
                <el-option
                  v-for="type in objectTypeList"
                  :key="type.value"
                  :label="type.label"
                  :value="type.value"
                />
              </el-select>
            </el-form-item>
            <el-form-item v-if="[1,2,3].includes(eventInfo.objectType)" style="margin-bottom: 21px" label="Biển số" prop="place">
              <el-input v-model="eventInfo.place" maxlength="9" />
            </el-form-item>
            <el-form-item v-else style="margin-bottom: 21px" label="Biển số">
              <el-input v-model="eventInfo.place" maxlength="9" />
            </el-form-item>
            <!-- <el-form-item style="margin-bottom: 21px" label="Ảnh" prop="image">
              <el-input
                v-model="eventInfo.image"
              />
            </el-form-item> -->
            <el-form-item label="Hình ảnh">
              <UploadImage
                :key="keyResetUpload"
                :list-image="listFile"
                @removeUploadImage="removeUploadImageHandle"
                @getListFile="getListFile"
              />
            </el-form-item>
            <el-form-item style="margin-bottom: 21px" label="Ghi chú" prop="description">
              <el-input v-model="eventInfo.description" type="textarea" rows="3" />
            </el-form-item>
          </div>
        </div>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button
          class="cancel-btn"
          type="info"
          @click="dialogEdit = false"
        >Hủy</el-button>
        <el-button
          type="primary"
          :loading="loadingVehicleEdit"
          @click="editEventHandle()"
        >Lưu
        </el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import axios from 'axios'
import UploadImage from '@/components/uploadImage'
import moment from 'moment'
import Cookies from 'js-cookie'
import user_default from '@/assets/images/no_preview.jpg'
import { inject } from 'vue'
import Search from '@/components/search-event'

export default {
  name: 'Events',
  components: {
    UploadImage,
    Search
  },
  setup() {
    const appName = inject('appName')
    console.log(appName)
  },
  filters: {
    formatDatetime: function(value) {
      if (!value) return ''
      return moment(value).format('DD/MM/YYYY HH:mm:ss')
    },
    getObjectType: function(value) {
      let type = 'Không xác định'
      if (!value) type = ''
      if (value === 1) type = 'Ô tô'
      if (value === 2) type = 'Xe máy'
      if (value === 3) type = 'Xe đạp điện'
      if (value === 4) type = 'Xe đạp'
      return type
    },
    getEventType: function(value) {
      let type = 'Không xác định'
      if (!value) type = ''
      if (value === 1) type = 'Vào'
      if (value === 2) type = 'Ra'
      return type
    },
    getSourceType: function(value) {
      let type = 'Không xác định'
      if (!value) type = ''
      if (value === 2) type = 'Nguồn thủ công'
      if (value === 1) type = 'Nguồn tự động'
      return type
    },
    getStatus: function(value) {
      let type = 'Không xác định'
      if (!value) type = ''
      if (value === 1) type = 'Chưa xem'
      if (value === 2) type = 'Xác minh sự kiện'
      if (value === 3) type = 'Đang xử lý'
      if (value === 4) type = 'Đã xử lý'
      if (value === 5) type = 'Báo sai'
      return type
    }
  },
  data() {
    return {
      loading: true,
      loadingVehicle: false,
      loadingVehicleEdit: false,
      eventList: [],
      multiSelected: [],
      allSelected: false,
      loading_delete_all: false,
      loading_add: false,
      dialogEdit: false,
      dialogAdd: false,
      flagKeySearch: 1,
      queryPage: {
        page: 0,
        size: 10,
        total: 0,
        search: '',
        startDate: '',
        endDate: '',
        objectType: [],
        eventType: [],
        sourceType: [],
        status: []
      },
      searchData: {
        startDate: '',
        endDate: '',
        objectType: [],
        eventType: [],
        sourceType: [],
        status: []
      },
      eventInfo: {
        uuid: '',
        eventType: '',
        place: '',
        objectType: '',
        sourceType: '',
        image: '',
        status: '',
        description: ''
      },
      eventDetail: {},
      imagecropperKey: 0,
	  keyResetUpload: 999,
      imagecropperShow: false,
      url: null,
      user_default: user_default,
      uploadReady: true,
      datePickerOptions: {
        disabledDate(date) {
          return date > moment().valueOf()
        }
      },
      file: {
        data: null
      },
      avatarHeight: 200,
      avatarWith: 200,
      flagUpload: 1,
      listFile: [],
      listFileVideo: [],
      eventTypeList: [
        {
          value: 1,
          label: 'Vào'
        },
        {
          value: 2,
          label: 'Ra'
        }
      ],
      objectTypeList: [
        {
          value: 1,
          label: 'Ô tô'
        },
        {
          value: 2,
          label: 'Xe máy'
        },
        {
          value: 3,
          label: 'Xe đạp điện'
        },
        {
          value: 4,
          label: 'Xe đạp'
        }
      ],
      sourceTypeList: [
        {
          value: 1,
          label: 'Nguồn tự động'
        },
        {
          value: 2,
          label: 'Nguồn thủ công'
        }
      ],
      statusList: [
        {
          value: 1,
          label: 'Chưa xem'
        },
        {
          value: 2,
          label: 'Xác minh sự kiện'
        },
        {
          value: 3,
          label: 'Đang xử lý'
        },
        {
          value: 4,
          label: 'Đã xử lý'
        },
        {
          value: 5,
          label: 'Báo sai'
        }
      ],
      rules: {
        eventType: [
          {
            required: true,
            message: 'Loại sự kiện là bắt buộc',
            trigger: 'blur'
          }
          //   { validator: vehicleType }
        ],
        objectType: [
          {
            required: true,
            message: 'Loại phương tiện là bắt buộc',
            trigger: 'blur'
          }
        ],
        place: [
          {
            required: true,
            message: 'Biển số là bắt buộc',
            trigger: 'blur'
          },
          { max: 9, message: 'Tối đa 9 ký tự', trigger: 'blur' }
        ]
      }
    }
  },
  watch: {
    multiSelected(newValue) {
      if (newValue.length === 0) {
        this.indeterminate = false
        this.allSelected = false
      } else if (newValue.length === this.eventList.length) {
        this.indeterminate = false
        this.allSelected = true
      } else {
        this.indeterminate = true
        this.allSelected = false
      }
    }
  },
  created() {
    this.getUser()
  },
  methods: {
    closeDialog(formName) {
      this.$refs[formName].clearValidate()
      this.$refs[formName].resetFields()
    },

    resetDialog() {
      this.url = null
      this.eventInfo = {
        uuid: '',
        eventType: '',
        place: '',
        objectType: '',
        sourceType: '',
        image: '',
        video: '',
        status: '',
        description: ''
      }
    },

    toggleAll(checked) {
      this.multiSelected = checked ? this.eventList.slice() : []
    },
    handleSizeChange(size) {
      this.queryPage.size = size
      this.getUser()
    },
    handleCurrentChange(page) {
      this.queryPage.page = page
      this.getUser()
    },
    onChangeInputSearch() {
      this.queryPage.page = 0
      this.getUser()
    },
    handleSelectionChange(val) {
      this.multiSelected = val
    },
    getListFile(listFile) {
      this.listFile = listFile
    },

    resetFilter() {
      this.flagKeySearch = Math.floor(Math.random() * 999999)
      this.queryPage = {
        page: 0,
        size: 10,
        startDate: moment().subtract(3, 'day').format('YYYY-MM-DDT00:00:00'),
        endDate: moment().endOf('day').format('YYYY-MM-DDTHH:mm:ss'),
        search: '',
        objectType: [],
        eventType: [],
        sourceType: []
      }
      this.closeDetail()
      this.getUser()
    },

    handleFilter() {
      if (this.queryPage.search) {
        this.queryPage.search = this.queryPage.search.trim()
      }
      this.queryPage.page = 0
      this.getUser()
    },

    onClickChild(value) {
      if (value) {
        this.queryPage.startDate = value.startDate
        this.queryPage.endDate = value.endDate
        this.queryPage.eventType = value.eventType && value.eventType.length > 0 ? [value.eventType] : []
        this.queryPage.sourceType = value.sourceType && value.sourceType.length > 0 ? [value.sourceType] : []
        this.queryPage.status = value.status
        this.queryPage.objectType = value.objectType
      }
      this.handleFilter()
    },

    getUser() {
      this.loading = true
      const headers = {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
	    const params = {
        page: this.queryPage.page > 0 ? this.queryPage.page - 1 : 0,
        size: this.queryPage.size,
        search: this.queryPage.search,
        startDate: this.queryPage.startDate == ''
          ? moment().subtract(3, 'day').format('YYYY-MM-DDT00:00:00')
          : moment(this.queryPage.startDate).format('YYYY-MM-DDTHH:mm:ss'),
        endDate: this.queryPage.endDate == ''
          ? moment(new Date()).format('YYYY-MM-DDT23:59:59')
          : moment(this.queryPage.endDate).format('YYYY-MM-DDTHH:mm:ss'),
        objectType: this.queryPage.objectType,
        eventType: this.queryPage.eventType,
        sourceType: this.queryPage.sourceType,
        status: this.queryPage.status
      }
      axios
        .post(process.env.VUE_APP_API + 'management', params, { headers })
        .then((res) => {
          if (res.data) {
            this.eventList = res.data.data
            this.queryPage.total = res.data.total
            this.loading = false
          }
        })
        .catch((err) => {
          console.log(moment(new Date()).format('YYYY-MM-DDT00:00:00'))
          this.loading = false
          this.$message({
            message: err.response.data.message,
            type: 'error'
          })
        })
    },

    closeDelete() {
      this.$refs.multipleTable.clearSelection()
      this.multipleSelection = []
    },

    destroyMulti() {
      this.$confirm('Bạn có chắc chắn muốn xóa lựa chọn này?', '', {
        confirmButtonText: 'Xác nhận',
        cancelButtonText: 'Thoát',
        type: 'warning'
      })
        .then(() => {
          const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          this.loading_delete_all = true
          let params = this.multiSelected.map(function(val) {
            return val.uuid
          })
          params = {
            uuids: params
          }
          axios
            .delete(process.env.VUE_APP_API + 'management/event', {
              headers: headers,
              data: params
            })
            .then((res) => {
              this.multiSelected = []
              this.onChangeInputSearch()
              this.loading_delete_all = false
              this.$message({
                type: 'success',
                message: res.data.message
              })
            })
            .catch((err) => {
              console.log(err)
              this.multiSelected = []
              this.onChangeInputSearch()
              this.loading_delete_all = false

              this.$message({
                message: err.response.data.message,
                type: 'error'
              })
            })
        })
        .catch(() => {})
    },

    updateEventStatus(uuid, status) {
      const params = {
        uuid: uuid,
        status: status
      }
      const headers = {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      this.loadingVehicleEdit = true
      axios
        .put(process.env.VUE_APP_API + 'management/update-status', params, { headers })
        .then((response) => {
          if (
            response.data.status === 200 ||
                response.data.status === 201
          ) {
            this.getUser()
            this.$message({
              message: response.data.message,
              type: 'success'
            })
            this.eventDetail = response.data.data
            if (this.eventDetail) {
              document.getElementById('vehicle-table').style.width =
            'calc(100% - 406px)'
              setTimeout(function() {
                document.getElementById('detail-vehicle').style.display = 'block'
                document.getElementById('detail-vehicle').style.width = '380px'
              }, 500)
            }
          } else {
            this.getUser()
            this.$message({
              message: response.data.message,
              type: 'error'
            })
          }
          this.loadingVehicleEdit = false
        })
        .catch((err) => {
          this.loadingVehicleEdit = false
          this.$message({
            message: err.response.data.message,
            type: 'error'
          })
        })
    },

    async rowClicked(row) {
      this.flag_active = row.uuid
      if (row) {
        this.eventDetail = row
        if (this.eventDetail.status == 1) {
          this.updateEventStatus(this.eventDetail.uuid, 2)
        } else {
          if (this.eventDetail) {
            document.getElementById('vehicle-table').style.width =
            'calc(100% - 406px)'
            setTimeout(function() {
              document.getElementById('detail-vehicle').style.display = 'block'
              document.getElementById('detail-vehicle').style.width = '380px'
            }, 500)
          }
        }
      } else {
        this.$message({
          message: 'Không tìm thấy phương tiện',
          type: 'error'
        })
        this.onChangeInputSearch()
      }
    },

    closeDetail() {
      document.getElementById('detail-vehicle').style.display = 'none'
      document.getElementById('detail-vehicle').style.width = '0'
      document.getElementById('vehicle-table').style.width = 'calc(100%)'
    },

    destroy(vehicle) {
      this.$confirm('Bạn có chắc chắn muốn xóa lựa chọn này?', '', {
        confirmButtonText: 'Xác nhận',
        cancelButtonText: 'Thoát',
        type: 'warning'
      })
        .then(() => {
          const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          this.loadingVehicle = true
          let params = this.multiSelected.map(function(val) {
            return val.uuid
          })
          params.push(vehicle.uuid)
          params = {
            uuids: params
          }
          axios
            .delete(process.env.VUE_APP_API + 'management/event', {
              headers: headers,
              data: params
            })
            .then((res) => {
              console.log(res)
              this.multiSelected = []
              this.onChangeInputSearch()
              this.closeDetail()
              this.loadingVehicle = false
			        this.$message({
                type: 'success',
                message: res.data.message
              })
            })
            .catch((err) => {
              console.log(err)
              this.multiSelected = []
              this.onChangeInputSearch()
              this.loadingVehicle = false

              this.$message({
                message: err.response.data.message,
                type: 'error'
              })
            })
        })
        .catch(() => {})
    },

    handleCreate() {
      this.resetDialog()

	  this.listFile = []
	  this.keyResetUpload = Math.random()
      this.dialogAdd = true
    },

    handleEdit(data) {
      this.eventInfo = _.cloneDeep(data)
	  console.log(this.eventInfo)
	  this.listFile = this.eventInfo.image ? this.eventInfo.image.split(',') : []
	  this.keyResetUpload = Math.random()
      this.dialogEdit = true
      this.$nextTick(() => {
        this.$refs['editForm'].clearValidate()
      })
    },

    editEvent() {
      this.eventInfo = this.$root.trimData(this.eventInfo)
      this.$refs.editForm.validate((valid) => {
        if (valid) {
          const params = {
            eventType: this.eventInfo.eventType,
            objectType: this.eventInfo.objectType,
            place: this.eventInfo.place,
            image: this.eventInfo.image,
            description: this.eventInfo.description
          }
          const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          this.loadingVehicleEdit = true
          axios
            .put(process.env.VUE_APP_API + 'management/event/' + this.eventInfo.uuid, params, { headers })
            .then((response) => {
              if (
                response.data.status === 200 ||
                response.data.status === 201
              ) {
                this.dialogEdit = false
                this.closeDetail()
                this.getUser()
                this.$message({
                  message: response.data.message,
                  type: 'success'
                })
                this.resetDialog()
              } else {
                this.dialogEdit = false
                this.getUser()
                this.$message({
                  message: response.data.message,
                  type: 'error'
                })
              }
              this.loadingVehicleEdit = false
              this.resetDialog()
            })
            .catch((err) => {
              this.loadingVehicleEdit = false
              this.$message({
                message: err.response.data.message,
                type: 'error'
              })
            })
        } else {
          return false
        }
      })
    },
    resetUploadFile() {
      this.uploadReady = false
      this.$nextTick(() => {
        this.uploadReady = true
      })
    },

    handleClick(e) {
      if (e.target !== this.$refs.file) {
        e.preventDefault()
        if (document.activeElement !== this.$refs) {
          this.$refs.file.click()
        }
      }
    },
    async editEventHandle() {
      if (this.listFile && this.listFile.length > 0) this.uploadImgEdit()
      if (
        this.listFile &&
          this.listFile.length == 0 &&
          this.listFileVideo &&
          this.listFileVideo.length > 0
      ) {
        this.uploadVideo()
      }
      if (
        this.listFile &&
          this.listFile.length == 0 &&
          this.listFileVideo &&
          this.listFileVideo.length == 0
      ) {
        this.editEvent()
      }
    },
    async addEventHandle() {
      if (this.listFile && this.listFile.length > 0) this.uploadImg()
      if (
        this.listFile &&
          this.listFile.length == 0 &&
          this.listFileVideo &&
          this.listFileVideo.length > 0
      ) {
        this.uploadVideo()
      }
      if (
        this.listFile &&
          this.listFile.length == 0 &&
          this.listFileVideo &&
          this.listFileVideo.length == 0
      ) {
        this.addEvent()
      }
    },
    async uploadVideo() {
      this.loadingImg = true
      const params = new FormData()
      this.listFileVideo.forEach((everyFile) => {
        params.append('file', everyFile.raw)
      })
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer '
      }
      axios
        .post(process.env.VUE_APP_UPLOAD_URL + 'file', params, {
          headers: headers
        })
        .then((res) => {
          if (res.data.data) {
            const data = []
            res.data.data.forEach((val) => {
              data.push(val.fileDownloadUri)
            })
            if (data.length > 0) this.eventInfo.videoUrl = data.toString()
            this.addEvent()
          }
        })
        .catch()
    },
    async uploadImgEdit() {
      this.loadingImg = true
      const params = new FormData()
      let flag = false
      this.listFile.forEach((everyFile) => {
        if (everyFile.raw) {
          flag = true
          params.append('file', everyFile.raw)
        }
      })
      let oldFile = this.listFile.filter(function(val) {
        return !val.raw
      })
      if (oldFile.length > 0) {
        oldFile = oldFile.map(function(val) {
          return val.url ? val.url : val
        })
      }
      console.log(oldFile)
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer '
      }
      if (flag) {
        axios
          .post(process.env.VUE_APP_UPLOAD_URL + 'file', params, {
            headers: headers
          })
          .then((res) => {
            if (res.data.data) {
              const data = oldFile
              res.data.data.forEach((val) => {
                data.push(val.fileDownloadUri)
              })
              if (data.length > 0) this.eventInfo.image = data.toString()
              if (this.listFileVideo && this.listFileVideo.length > 0) {
                this.uploadVideo()
              }
              if (this.listFileVideo && this.listFileVideo.length == 0) {
                this.editEvent()
              }
            }
          })
          .catch()
      } else {
        if (oldFile.length > 0) this.eventInfo.imageUrl = oldFile.toString()
        if (this.listFileVideo && this.listFileVideo.length > 0) {
          this.uploadVideo()
        }
        if (this.listFileVideo && this.listFileVideo.length == 0) {
          this.editEvent(this.eventInfo)
        }
      }
    },
    async uploadImg() {
      this.loadingImg = true
      const params = new FormData()
      let flag = false
      this.listFile.forEach((everyFile) => {
        if (everyFile.raw) {
          flag = true
          params.append('file', everyFile.raw)
        }
      })
      let oldFile = this.listFile.filter(function(val) {
        return !val.raw
      })
      if (oldFile.length > 0) {
        oldFile = oldFile.map(function(val) {
          return val.url ? val.url : val
        })
      }
      console.log(oldFile)
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer '
      }
      if (flag) {
        axios
          .post(process.env.VUE_APP_UPLOAD_URL + 'file', params, {
            headers: headers
          })
          .then((res) => {
            if (res.data.data) {
              const data = oldFile
              res.data.data.forEach((val) => {
                data.push(val.fileDownloadUri)
              })
              if (data.length > 0) this.eventInfo.image = data.toString()
              if (this.listFileVideo && this.listFileVideo.length > 0) {
                this.uploadVideo()
              }
              if (this.listFileVideo && this.listFileVideo.length == 0) {
                this.addEvent()
              }
            }
          })
          .catch()
      } else {
        if (oldFile.length > 0) this.eventInfo.imageUrl = oldFile.toString()
        if (this.listFileVideo && this.listFileVideo.length > 0) {
          this.uploadVideo()
        }
        if (this.listFileVideo && this.listFileVideo.length == 0) {
          this.addEvent(this.eventInfo)
        }
      }
    },

    addEvent() {
      this.eventInfo = this.$root.trimData(this.eventInfo)
      this.$refs.addForm.validate((valid) => {
        if (valid) {
          const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          const data = _.cloneDeep(this.eventInfo)
          // const paramRegister = this.eventInfo
          const paramRegister = {
            eventType: data.eventType,
            objectType: data.objectType,
            place: data.place,
            description: data.description,
            image: this.eventInfo.image
          }
          this.loading_add = true
          axios
            .post(process.env.VUE_APP_API + 'management/event', paramRegister, { headers })
            .then((response) => {
              console.log(response.data)
              if (response.data.status === 200 || response.data.status === 201) {
                this.$message({
                  type: 'success',
                  message: response.data.message
                })
                this.dialogAdd = false
                this.getUser()
                this.loading_add = false
                this.resetDialog()
              } else {
                this.$message({
                  message: response.data.message,
                  type: 'error'
                })
                this.dialogAdd = false
                this.loading_add = false
                this.getUser()
                this.resetDialog()
              }
            })
            .catch((err) => {
              this.loading_add = false
              this.$message({
                message: err.response.data.message,
                type: 'error'
              })
            })
        } else {
          return false
        }
      })
    },

    removeUploadImageHandle(file) {
      this.listFile = this.listFile.filter(function(val) {
        return val != file
      })
    }
  }
}
</script>

<style lang="scss">
.el-dialog {
  margin-top: 30px !important;
}

.item-left {
  width: 48%;
  float: left;
  margin-right: 2%;
}

.item-right {
  width: 48%;
  float: left;
  margin-left: 2%;
}

.block-item {
  width: 100%;
  float: left;
}

#select-all-delete .el-icon-close {
  font-size: 14px;
  margin-left: 18px;
  margin-right: 29px;
  top: 4px;
  cursor: pointer;
}

.el-icon-camera-solid {
  font-size: 25px;
  color: #9a9a9a;
  position: relative;
  top: -50px;
  left: 54px;
  background: #dfdfdf;
  border-radius: 50%;
  padding: 8px;
  border: 1px solid white;
}
</style>
