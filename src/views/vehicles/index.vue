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
          :data="vehicleList"
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
          <el-table-column label="Loại phương tiện">
            <template slot-scope="scope">{{
              scope.row.vehicleType | getVehicleTypeName
            }}</template>
          </el-table-column>
          <el-table-column prop="place" label="Biển số" />
          <el-table-column prop="color" label="Màu sắc" />
          <el-table-column prop="brand" label="Thương hiệu" />
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
        <template v-if="vehicleDetail">
          <div class="avatar" />

          <template v-if="vehicleDetail.vehicleType">
            <div class="item-label">Loại phương tiện</div>
            <div class="item-text">
              {{ vehicleDetail.vehicleType | getVehicleTypeName }}
            </div>
          </template>

          <template v-if="vehicleDetail.brand">
            <div class="item-label">Thương hiệu</div>
            <div class="item-text">{{ vehicleDetail.brand }}</div>
          </template>

          <template v-if="vehicleDetail.color">
            <div class="item-label">Màu sắc</div>
            <div class="item-text">{{ vehicleDetail.color }}</div>
          </template>

          <template v-if="vehicleDetail.place">
            <div class="item-label">Biển số</div>
            <div class="item-text">{{ vehicleDetail.place }}</div>
          </template>

          <template v-if="vehicleDetail.ownerName">
            <div class="item-label">Chủ sở hữu</div>
            <div class="item-text">{{ vehicleDetail.ownerName }}</div>
          </template>

          <div class="action">
            <el-button
              type="info"
              @click="handleEdit(vehicleDetail)"
            ><i class="el-icon-edit" /> Sửa
            </el-button>
            <el-button
              type="danger"
              style="float: right"
              :loading="loadingVehicle"
              @click="destroy(vehicleDetail)"
            ><i class="el-icon-delete" /> Xóa
            </el-button>
          </div>
        </template>
      </div>
    </div>

    <el-dialog
      title="Cập nhật"
      :visible.sync="dialogEdit"
      width="500px"
      :close-on-click-modal="false"
    >
      <el-form
        ref="editForm"
        :model="vehicleInfo"
        :rules="rules"
        label-position="top"
        label-width="200px"
      >
        <div class="block-item">
          <div class="item-mid">
            <el-form-item style="margin-bottom: 21px" label="Loại phương tiện" prop="vehicleType">
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
            <el-form-item style="margin-bottom: 21px" label="Thương hiệu" prop="brand">
              <el-input v-model="vehicleInfo.brand" />
            </el-form-item>
            <el-form-item style="margin-bottom: 21px" label="Màu sắc" prop="color">
              <el-input
                v-model="vehicleInfo.color"
              />
            </el-form-item>
            <el-form-item style="margin-bottom: 21px" label="Biển số" prop="place">
              <el-input v-model="vehicleInfo.place" maxlength="9" />
            </el-form-item>
          </div>
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button class="cancel-btn" type="info" @click="dialogEdit = false">Hủy</el-button>
        <el-button
          type="primary"
          :loading="loadingVehicleEdit"
          @click="editVehicle()"
        >Lưu
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import Cookies from 'js-cookie'

export default {
  name: 'Vehicles',
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
    return {
      loading: true,
      loadingVehicle: false,
      loadingVehicleEdit: false,
      vehicleList: [],
      multiSelected: [],
      allSelected: false,
      loading_delete_all: false,
      dialogEdit: false,
      queryPage: {
        page: 0,
        size: 10,
        total: 0,
        search: ''
      },
      vehicle: {
        uuid: '',
        vehicleType: '',
        place: '',
        color: '',
        brand: '',
        ownerName: '',
        status: null,
        ownerId: ''
      },
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
    this.getVehicle()
  },
  methods: {
    closeDialog(formName) {
      this.$refs[formName].clearValidate()
      this.$refs[formName].resetFields()
    },
    toggleAll(checked) {
      this.multiSelected = checked ? this.userList.slice() : []
    },
    handleSizeChange(size) {
      this.queryPage.size = size
      this.getVehicle()
    },
    handleCurrentChange(page) {
      this.queryPage.page = page
      this.getVehicle()
    },
    onChangeInputSearch() {
      this.queryPage.page = 0
      this.getVehicle()
    },
    handleSelectionChange(val) {
      this.multiSelected = val
    },
    getVehicle() {
      this.loading = true
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      axios
        .get(process.env.VUE_APP_API + 'vehicle', {
          headers: headers,
          params: {
            page: this.queryPage.page > 0 ? this.queryPage.page - 1 : 0,
            size: this.queryPage.size,
            search: this.queryPage.search
          }
        })
        .then((res) => {
          if (res.data) {
            this.vehicleList = res.data.data
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
            .delete(process.env.VUE_APP_API + 'vehicle', {
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

    async rowClicked(row) {
      this.flag_active = row.uuid
      if (row) {
        this.vehicleDetail = row
        if (this.vehicleDetail) {
          document.getElementById('vehicle-table').style.width =
            'calc(100% - 425px)'
          setTimeout(function() {
            document.getElementById('detail-vehicle').style.display = 'block'
            document.getElementById('detail-vehicle').style.width = '400px'
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
            .delete(process.env.VUE_APP_API + 'vehicle', {
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

    handleEdit(data) {
      this.vehicleInfo.uuid = data.uuid
      this.vehicleInfo.vehicleType = data.vehicleType
      this.vehicleInfo.place = data.place
      this.vehicleInfo.color = data.color
      this.vehicleInfo.brand = data.brand
      this.vehicleInfo.ownerName = data.ownerName
      this.vehicleInfo.status = data.status
      this.vehicleInfo.ownerId = data.ownerId
      this.dialogEdit = true
      this.$nextTick(() => {
        this.$refs['editForm'].clearValidate()
      })
    },

    editVehicle() {
    //   this.vehicleInfo = this.$root.trimData(this.vehicleInfo)
      this.$refs.editForm.validate((valid) => {
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
          this.loadingVehicleEdit = true
          axios
            .put(process.env.VUE_APP_API + 'vehicle/' + this.vehicleInfo.uuid, params, { headers })
            .then((response) => {
              if (response.data.status === 200 || response.data.status === 201) {
                this.dialogEdit = false
                this.closeDetail()
                this.getVehicle()
                this.$message({
                  message: response.data.message,
                  type: 'success'
                })
              } else {
                this.dialogEdit = false
                this.getVehicle()
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
        } else {
          return false
        }
      })
    }

  }
}
</script>

<style>
</style>
