<!-- eslint-disable vue/valid-v-model -->
<template>
  <div class="body">
    <div>
      <el-input
        v-model="queryPage.search"
        class="input-search"
        placeholder="Tìm kiếm"
        prefix-icon="el-icon-search"
        @change="onChangeInputSearch"
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
          :data="userList"
          style="width: 100%"
          @selection-change="handleSelectionChange"
          @row-click="rowClicked"
        >
          <el-table-column type="selection" width="70" />
          <el-table-column label="STT" width="70">
            <template slot-scope="scope">{{
              scope.$index +
                1 +
                (queryPage.page > 0 ? queryPage.page - 1 : 0) * queryPage.size
            }}</template>
          </el-table-column>
          <el-table-column prop="fullName" label="Họ và tên" />
          <el-table-column prop="phoneNumber" label="Số điện thoại" />
          <el-table-column prop="address" label="Địa chỉ" />
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
        <template v-if="userDetail">
          <div class="avatar" />

          <template v-if="userDetail.fullName">
            <div class="item-label">Họ và tên</div>
            <div class="item-text">
              {{ userDetail.fullName }}
            </div>
          </template>

          <template v-if="userDetail.phoneNumber">
            <div class="item-label">Số điện thoại</div>
            <div class="item-text">{{ userDetail.phoneNumber }}</div>
          </template>

          <template v-if="userDetail.address">
            <div class="item-label">Địa chỉ</div>
            <div class="item-text">{{ userDetail.address }}</div>
          </template>

          <div class="action">
            <el-button
              type="info"
              @click="handleEdit(userDetail)"
            ><i class="el-icon-edit" /> Sửa
            </el-button>
            <el-button
              type="primary"
              style="float: right; width: 170px"
              :loading="loadingVehicle"
              @click="handleAddVehicle(userDetail)"
            ><i class="el-icon-plus" /> Thêm phương tiện
            </el-button>
          </div>
          <div class="action" style="margin-top: 15px">
            <el-button
              type="danger"
              :loading="loadingVehicle"
              @click="destroy(userDetail)"
            ><i class="el-icon-delete" /> Xóa
            </el-button>
            <el-button
              type="default"
              style="float: right; width: 170px; background-color: #d3d3d3"
              @click="viewVehicle(userDetail)"
            ><i class="el-icon-view" /> Xem phương tiện
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
        :model="userInfo"
        :rules="rules"
        label-position="top"
        label-width="200px"
      >
        <div class="block-item">
          <div class="item-mid">
            <el-form-item
              style="margin-bottom: 21px"
              label="Họ và tên"
              prop="fullName"
            >
              <el-input
                v-model="userInfo.fullName"
                placeholder="Nhập họ và tên"
              />
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Số điện thoại"
              prop="phoneNumber"
            >
              <el-input
                v-model="userInfo.phoneNumber"
                placeholder="Nhập số điện thoại"
              />
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Địa chỉ"
              prop="address"
            >
              <el-input
                v-model="userInfo.address"
                type="textarea"
                :rows="3"
                placeholder="Nhập địa chỉ"
                maxlength="255"
              />
            </el-form-item>
          </div>
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button
          class="cancel-btn"
          type="info"
          @click="dialogAdd = false"
        >Hủy</el-button>
        <el-button
          type="primary"
          :loading="loadingVehicle"
          @click="addOwner()"
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
        :model="userInfo"
        :rules="rules"
        label-position="top"
        label-width="200px"
      >
        <div class="block-item">
          <div class="item-mid">
            <el-form-item
              style="margin-bottom: 21px"
              label="Họ và tên"
              prop="fullName"
            >
              <el-input
                v-model="userInfo.fullName"
                placeholder="Nhập họ và tên"
              />
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Số điện thoại"
              prop="phoneNumber"
            >
              <el-input
                v-model="userInfo.phoneNumber"
                placeholder="Nhập số điện thoại"
              />
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Địa chỉ"
              prop="address"
            >
              <el-input
                v-model="userInfo.address"
                type="textarea"
                :rows="3"
                placeholder="Nhập địa chỉ"
                maxlength="255"
              />
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
          :loading="loadingVehicle"
          @click="editUser()"
        >Lưu
        </el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="Thêm phương tiện"
      :visible.sync="dialogAddVehicle"
      width="500px"
      :close-on-click-modal="false"
      @close="closeDialog('addVehicleForm')"
    >
      <el-form
        ref="addVehicleForm"
        :model="vehicleInfo"
        :rules="rulesVehicle"
        label-position="top"
        label-width="200px"
      >
        <div class="block-item">
          <div class="item-mid">
            <el-form-item
              style="margin-bottom: 21px"
              label="Loại phương tiện"
              prop="vehicleType"
            >
              <el-select
                v-model="vehicleInfo.vehicleType"
                placeholder="Chọn loại phương tiện"
                style="width: 100%"
              >
                <el-option
                  v-for="type in vehicleTypeList"
                  :key="type.value"
                  :label="type.label"
                  :value="type.value"
                />
              </el-select>
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Thương hiệu"
              prop="brand"
            >
              <el-input v-model="vehicleInfo.brand" />
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Màu sắc"
              prop="color"
            >
              <el-input v-model="vehicleInfo.color" />
            </el-form-item>
            <el-form-item
              style="margin-bottom: 21px"
              label="Biển số"
              prop="place"
            >
              <el-input v-model="vehicleInfo.place" maxlength="9" />
            </el-form-item>
          </div>
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button
          class="cancel-btn"
          type="info"
          @click="dialogAddVehicle = false"
        >Hủy</el-button>
        <el-button
          type="primary"
          :loading="loadingAddVehicle"
          @click="addVehicle()"
        >Lưu
        </el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="Danh sách phương tiện"
      :visible.sync="dialogView"
      width="700px"
      :close-on-click-modal="false"
    >
      <el-table :data="vehicleList">
        <el-table-column label="STT" width="70">
          <template slot-scope="scope">{{
            scope.$index + 1
          }}</template>
        </el-table-column>
        <el-table-column label="Loại phương tiện">
          <template slot-scope="scope">{{
            scope.row.vehicleType | getVehicleTypeName
          }}</template>
        </el-table-column>
        <el-table-column prop="brand" label="Thương hiệu" />
        <el-table-column prop="color" label="Màu sắc" />
        <el-table-column prop="place" label="Biển số" />
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
import Cookies from 'js-cookie'
import { validPhone } from '@/utils/validate'
import { inject } from 'vue'

export default {
  setup() {
    const appName = inject('appName')
    console.log(appName)
  },
  name: 'Owners',
  filters: {
    getVehicleTypeName: function(value) {
      let type = ''
      if (!value) type = ''
      if (value === 'CAR') type = 'Ô tô'
      if (value === 'MOTO') type = 'Xe máy'
      if (value === 'TRAM') type = 'Xe đạp điện'
      if (value === 'BIKE') type = 'Xe đạp'
      return type
    }
  },
  data() {
    const validateMobile = (rule, value, callback) => {
      if (value && !validPhone(value)) {
        callback(new Error('Vui lòng nhập đúng định dạng số điện thoại'))
      } else {
        callback()
      }
    }

    return {
      loading: true,
      loadingVehicle: false,
	  loadingAddVehicle: false,
      userList: [],
	  vehicleList: [],
      multiSelected: [],
      allSelected: false,
      loading_delete_all: false,
      loading_add: false,
      dialogEdit: false,
      dialogAdd: false,
	  dialogAddVehicle: false,
	  dialogView: false,
      queryPage: {
        page: 0,
        size: 10,
        total: 0,
        search: ''
      },
      user: {
        uuid: '',
        fullName: '',
        phoneNumber: '',
        address: ''
      },
      userInfo: {
        uuid: '',
        fullName: '',
        phoneNumber: '',
        address: ''
      },
      userUpdate: {
        uuid: '',
        fullName: '',
        phoneNumber: '',
        address: ''
      },
      userDetail: {},
      vehicleInfo: {
        uuid: '',
        vehicleType: '',
        place: '',
        color: '',
        brand: '',
        ownerName: '',
        status: '',
        ownerId: ''
      },
      vehicleDetail: {},
	  vehicleTypeList: [
        {
          value: 'CAR',
          label: 'Ô tô'
        },
        {
          value: 'MOTO',
          label: 'Xe máy'
        },
        {
          value: 'TRAM',
          label: 'Xe đạp điện'
        },
        {
          value: 'BIKE',
          label: 'Xe đạp'
        }
	  ],
      rules: {
        fullName: [
          {
            required: true,
            message: 'Họ và tên là bắt buộc',
            trigger: 'blur'
          }
          //   { validator: vehicleType }
        ],
        phoneNumber: [
          {
            required: true,
            message: 'Số điện thoại là bắt buộc',
            trigger: 'blur'
          },
          { validator: validateMobile }
        ],
        address: [
          {
            required: true,
            message: 'Địa chỉ là bắt buộc',
            trigger: 'blur'
          },
          { max: 255, message: 'Tối đa 255 ký tự', trigger: 'blur' }
        ]
      },
	  rulesVehicle: {
        vehicleType: [
          {
            required: true,
            message: 'Loại phương tiện là bắt buộc',
            trigger: 'blur'
          }
        //   { validator: vehicleType }
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
      } else if (newValue.length === this.userList.length) {
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
      this.userInfo = {
        uuid: '',
        phoneNumber: '',
        fullName: '',
        address: ''
      }
    },

    toggleAll(checked) {
      this.multiSelected = checked ? this.userList.slice() : []
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
    getUser() {
      this.loading = true
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      axios
        .get(process.env.VUE_APP_API + 'vehicle/owner', {
          headers: headers,
          params: {
            page: this.queryPage.page > 0 ? this.queryPage.page - 1 : 0,
            size: this.queryPage.size,
            search: this.queryPage.search
          }
        })
        .then((res) => {
          if (res.data) {
            this.userList = res.data.data
            this.queryPage.total = res.data.total
            this.loading = false
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
            .delete(process.env.VUE_APP_API + 'vehicle/owner', {
              headers: headers,
              data: params
            })
            .then((res) => {
              this.multiSelected = []
              this.onChangeInputSearch()
              this.loading_delete_all = false
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

    async rowClicked(row) {
      this.flag_active = row.uuid
      if (row) {
        this.userDetail = row
        if (this.userDetail) {
          document.getElementById('vehicle-table').style.width =
            'calc(100% - 406px)'
          setTimeout(function() {
            document.getElementById('detail-vehicle').style.display = 'block'
            document.getElementById('detail-vehicle').style.width = '380px'
          }, 500)
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
            .delete(process.env.VUE_APP_API + 'vehicle/owner', {
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
      this.dialogAdd = true
	  this.userInfo.gender = 0
    },

    handleEdit(data) {
      this.userInfo = _.cloneDeep(data)
      this.dialogEdit = true
      this.$nextTick(() => {
        this.$refs['editForm'].clearValidate()
      })
    },

    editUser() {
      this.userInfo = this.$root.trimData(this.userInfo)
      this.$refs.editForm.validate((valid) => {
        if (valid) {
          const params = {
            fullName: this.userInfo.fullName.trim(),
            phoneNumber: this.userInfo.phoneNumber.trim(),
            address: this.userInfo.address.trim()
          }
          const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          this.loadingVehicle = true
          axios
            .put(process.env.VUE_APP_API + 'vehicle/owner/' + this.userInfo.uuid, params, { headers })
            .then((response) => {
              if (
                response.data.status === 200 ||
                response.data.status === 201
              ) {
                this.dialogEdit = false
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
              this.loadingVehicle = false
              this.resetDialog()
            })
            .catch((err) => {
              this.loadingVehicle = false
              console.log(err)
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
    handleClick(e) {
      if (e.target !== this.$refs.file) {
        e.preventDefault()
        if (document.activeElement !== this.$refs) {
          this.$refs.file.click()
        }
      }
    },

    addOwner() {
      this.userInfo = this.$root.trimData(this.userInfo)
      this.$refs.addForm.validate((valid) => {
        if (valid) {
          const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          const data = _.cloneDeep(this.userInfo)
          // const paramRegister = this.userInfo
          const paramRegister = {
            phoneNumber: data.phoneNumber,
            fullName: data.fullName,
            address: data.address
          }
          this.loading_add = true
          axios
            .post(process.env.VUE_APP_API + 'vehicle/owner', paramRegister, { headers })
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

    handleAddVehicle(data) {
      this.vehicleInfo.ownerId = data.uuid
      this.dialogAddVehicle = true
      this.$nextTick(() => {
        this.$refs['addVehicleForm'].clearValidate()
      })
    },

    addVehicle() {
    //   this.vehicleInfo = this.$root.trimData(this.vehicleInfo)
      this.$refs.addVehicleForm.validate((valid) => {
        if (valid) {
          const params = {
            vehicleType: this.vehicleInfo.vehicleType.trim(),
            place: this.vehicleInfo.place.trim(),
            color: this.vehicleInfo.color.trim(),
            brand: this.vehicleInfo.brand.trim(),
            owner: this.vehicleInfo.ownerId
          }
    	  const headers = {
            'Content-Type': 'application/json',
            Authorization: 'Bearer ' + Cookies.get('access-token')
          }
          this.loadingAddVehicle = true
          axios
            .post(process.env.VUE_APP_API + 'vehicle', params, { headers })
            .then((response) => {
              console.log(response.data.data)
              if (response.data.status === 200 || response.data.status === 201) {
                this.dialogAddVehicle = false
                this.closeDetail()
                this.getUser()
                this.$message({
                  message: response.data.message,
                  type: 'success'
                })
                this.loadingAddVehicle = false
              } else {
                this.dialogAddVehicle = false
                this.getUser()
                this.$message({
                  message: response.data.message,
                  type: 'error'
                })
              }
              this.loadingAddVehicle = false
            })
            .catch((err) => {
              this.$message({
                message: err.response.data.message,
                type: 'error'
              })
              this.loadingAddVehicle = false
            })
        } else {
          return false
        }
      })
    },

    async viewVehicle(data) {
	  const headers = {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      axios
        .get(process.env.VUE_APP_API + 'vehicle/owner/' + data.uuid, { headers })
        .then((response) => {
          if (response.data.status === 200 || response.data.status === 201) {
            this.vehicleList = response.data.data
            this.dialogView = true
            console.log(this.vehicleList)
          } else {
            this.$message({
              message: response.data.message,
              type: 'error'
            })
          }
        })
        .catch((err) => {
          this.$message({
            message: err.response.data.message,
            type: 'error'
          })
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
  margin-left: 28px;
  margin-right: 38px;
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
