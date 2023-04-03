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
        @click="dialogAdd = true"
      >Thêm mới</el-button>
    </div>
    <b-card class="mb-4" no-body>
      <!-- <div class="table-responsive"> -->
      <div v-if="multiSelected.length > 0" id="select-all-delete">
        <i class="el-icon-close" @click="closeDelete()" />
        <span v-if="multiSelected.length < 10">0{{ multiSelected.length }}</span><span v-else>{{ multipleSelection.length }}</span> mục được chọn
        <el-button
          type="danger"
          :loading="loading_delete_all"
          @click="destroyMulti()"
        ><i class="el-icon-delete" /> Xóa</el-button>
      </div>
      <el-table
        ref="multipleTable"
        :data="userList"
        style="width: 100%"
        @selection-change="handleSelectionChange"
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
        <el-table-column prop="username" label="Tên đăng nhập" />
        <el-table-column label="Giới tính">
          <template slot-scope="scope">{{
            scope.row.gender === 1 ? "Nam" : "Nữ"
          }}</template>
        </el-table-column>
        <el-table-column label="Ngày sinh">
          <template slot-scope="scope">{{
            scope.row.birthday | formatDatetime
          }}</template>
        </el-table-column>
        <el-table-column prop="phoneNumber" label="Số điện thoại" />
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
      <!-- </div> -->
    </b-card>

    <el-dialog
      title="Thêm mới"
      :visible.sync="dialogAdd"
      width="700px"
      :close-on-click-modal="false"
      @close="closeDialog('addForm')"
    >
      <el-form
        ref="addForm"
        :model="user"
        :rules="rules"
        label-position="top"
        label-width="100%"
      >
        <div class="block-item">
          <div class="item-left">
            <el-form-item
              style="margin-bottom: 21px"
              label="Tên đăng nhập"
              prop="username"
            >
              <el-input v-model.trim="user.username" />
            </el-form-item>

            <el-form-item
              style="margin-bottom: 21px"
              label="Mật khẩu đăng nhập"
              prop="password"
            >
              <el-input v-model="user.password" type="password" />
            </el-form-item>

            <el-form-item
              style="margin-bottom: 21px"
              label="Nhập lại mật khẩu"
              prop="matchingPassword"
            >
              <el-input v-model="user.matchingPassword" type="password" />
            </el-form-item>
          </div>
          <div class="item-right">
            <el-form-item>
              <div class="img_avatar" label="Avatar" prop="avatar">
                <img v-if="url" class="image" height="178px" width="178px">
                <img v-else class="image" height="178px" width="178px">
                <!-- <img v-else class="image" @click="handleClick"> -->
                <div class="img_icon">
                  <input
                    v-if="uploadReady"
                    ref="file"
                    type="file"
                    hidden
                    @change="fileSelected"
                  >
                  <div class="avatar-uploader">
                    <i class="el-icon-camera-solid avatar-uploader-icon" />
                  </div>
                </div>
              </div>
            </el-form-item>
          </div>
        </div>
        <div class="block-item">
          <div class="item-left">
            <el-form-item
              style="margin-bottom: 21px"
              label="Họ tên"
              prop="fullName"
            >
              <el-input v-model="user.fullName" />
            </el-form-item>

            <el-form-item
              style="margin-bottom: 21px"
              label="Ngày sinh"
              prop="birthday"
            >
              <el-date-picker
                v-model="user.birthday"
                type="date"
                placeholder="Ngày sinh"
                format="dd/MM/yyyy"
                style="width: 100%"
              />
            </el-form-item>

            <el-form-item
              style="margin-bottom: 21px"
              label="Email"
              prop="email"
            >
              <el-input v-model="user.email" />
            </el-form-item>

            <el-form-item
              style="margin-bottom: 21px"
              label="Điện thoại"
              prop="phoneNumber"
            >
              <el-input v-model="user.phoneNumber" />
            </el-form-item>
          </div>
          <div class="item-right" />
        </div>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button type="info" @click="dialogAdd = false">Thoát</el-button>
        <el-button type="primary">Lưu </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
import { validEmail, validPhone } from '@/utils/validate'
import Cookies from 'js-cookie'

export default {
  name: 'Accounts',
  filters: {
    formatDatetime: function(value) {
      if (!value) return ''
      return moment(value, 'YYYY-MM-DD').format('DD/MM/YYYY')
    }
  },
  data() {
    const validateEmail = (rule, value, callback) => {
      if (!validEmail(value)) {
        callback(new Error('Vui lòng nhập đúng định dạng email'))
      } else {
        callback()
      }
    }

    const validateMobile = (rule, value, callback) => {
      if (value && !validPhone(value)) {
        callback(new Error('Vui lòng nhập đúng định dạng số điện thoại'))
      } else {
        callback()
      }
    }

    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('Xác nhận mật khẩu là bắt buộc'))
      } else if (value !== this.userInfo.password) {
        callback(new Error('Mật khẩu không trùng khớp'))
      } else {
        callback()
      }
    }

    return {
      userList: [],
      multiSelected: [],
      allSelected: false,
      loading_delete_all: false,
      queryPage: {
        page: 0,
        size: 10,
        total: 0,
        search: ''
      },
      dialogAdd: false,
      user: {
        username: '',
        password: '',
        matchingPassword: '',
        fullName: '',
        phoneNumber: '',
        email: '',
        gender: null,
        birthday: '',
        address: '',
        avatar: '',
        role: ''
      },
      rules: {
        username: [
          {
            required: true,
            message: 'Tên đăng nhập là bắt buộc',
            trigger: 'blur'
          }
        ],
        fullName: [
          { required: true, message: 'Họ và tên là bắt buộc', trigger: 'blur' }
        ],
        phoneNumber: [
          {
            required: true,
            message: 'Số điện thoại là bắt buộc',
            trigger: 'blur'
          },
          { validator: validateMobile }
        ],
        email: [
          { required: true, message: 'Email là bắt buộc', trigger: 'blur' },
          { validator: validateEmail }
        ],
        password: [
          { required: true, message: 'Mật khẩu là bắt buộc', trigger: 'blur' },
          { min: 4, message: 'Mật khẩu tối thiểu 4 ký tự', trigger: 'blur' }
        ],
        matchingPassword: [
          {
            required: true,
            message: 'Nhập lại mật khẩu là bắt buộc',
            trigger: 'blur'
          },
          { validator: validatePass2, trigger: 'blur' }
        ],
        birthday: [
          { required: true, message: 'Ngày sinh là bắt buộc', trigger: 'blur' }
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
      const headers = {
        'Content-Type': 'multipart/form-data',
        Authorization: 'Bearer ' + Cookies.get('access-token')
      }
      axios
        .get(process.env.VUE_APP_API + 'user', {
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
          }
        })
        .catch((err) => {
          console.log(err)
          this.loading = false
          this.$message({
            type: 'error',
            message: 'Có lỗi xảy ra vui lòng thử lại sau!'
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
            .delete(process.env.VUE_APP_API + 'user', {
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
                type: 'warning',
                message: 'Có lỗi xảy ra vui lòng thử lại'
              })
            })
        })
        .catch(() => {})
    }
  }
}
</script>

<style>
</style>
