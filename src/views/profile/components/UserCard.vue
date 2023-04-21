<template>
  <el-card style="width: 550px;">
    <div class="profile-editor-container">
      <div class="chart-wrapper update-password-block">
        <div class="avatar">
          <div class="header">
            <h2>Trang cá nhân</h2>
          </div>
          <div class="body">
            <!-- <img v-if="user.avatar" :src="user.avatar" alt="avatar">
          <img v-else :src="imgUserDefault" alt="avatar default"> -->

            <pan-thumb v-if="userInfo.avatar" :image="userInfo.avatar" :height="'100px'" :width="'100px'" :hoverable="false">
              {{ userInfo.role === 1 ? 'Quản trị viên' : 'Nhân viên' }}
            </pan-thumb>
            <pan-thumb v-else :image="user.avatar" :height="'100px'" :width="'100px'" :hoverable="false">
              {{ userInfo.role === 1 ? 'Quản trị viên' : 'Nhân viên' }}
            </pan-thumb>
          </div>
        </div>
        <div class="general-info">
          <div class="body">
            <p v-if="user.fullName"><span class="label">Họ và tên </span> <span class="value">{{ user && user.fullName }}</span></p>
            <p v-if="user.userName"><span class="label">Tài khoản </span> <span class="value">{{ user && user.userName }}</span></p>
            <p v-if="user.email"><span class="label">Email </span> <span class="value">{{ user && user.email }}</span></p>
            <p v-if="user.mobile"><span class="label">Số điện thoại </span> <span class="value">{{ user && user.mobile }}</span></p>
          </div>
        </div>
        <!-- <h2>Thông tin cá nhân</h2>
      <p><b>Họ và tên: </b> {{ user && user.fullName }}</p>
      <p><b>Tài khoản: </b>{{ user && user.userName }}</p>
      <p><b>Email: </b>{{ user && user.email }}</p>
      <p><b>Số điện thoại: </b>{{ user && user.mobile }}</p> -->
        <ElcButton
          class="btn-center"
          size="small"
          type="primary"
          @click="handleUpdatePassword()"
        >Đổi mật khẩu</ElcButton>
      </div>
      <div class="update-password">
        <el-dialog
          title="Đổi mật khẩu"
          :visible.sync="dialogUpdate"
          width="650px"
          :close-on-click-modal="false"
          @close="closeDialog('updateForm')"
        >
          <el-form
            ref="updateForm"
            :model="userInfo"
            :rules="rules"
            label-position="top"
            label-width="100%"
          >
            <div class="block-item">
              <el-form-item label="Mật khẩu hiện tại" prop="curentPassword">
                <el-input v-model="userInfo.curentPassword" maxlength="32" type="password" />
              </el-form-item>

              <el-form-item label="Mật khẩu mới" prop="newPassword">
                <el-input v-model="userInfo.newPassword" maxlength="32" type="password" />
              </el-form-item>

              <el-form-item label="Xác nhận mật khẩu" prop="rePassword">
                <el-input
                  v-model="userInfo.rePassword"
                  type="password"
                  maxlength="32"
                />
              </el-form-item>
            </div>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button class="btn-cancel" type="danger" @click="dialogUpdate = false">Hủy</el-button>
            <ElcButton
              size="small"
              type="primary"
              :loading="loading"
              @click="updatePassword()"
            >Cập nhật</ElcButton></div>
        </el-dialog>
      </div>
    </div>
  </el-card>
</template>

<script>
import PanThumb from '@/components/PanThumb'
import Cookies from 'js-cookie'

export default {
  components: { PanThumb },
  props: {
    user: {
      type: Object,
      default: () => {
        return {
          name: '',
          email: '',
          avatar: '',
          role: ''

          // username: '',
          // address: '',
          // email: '',
          // avatar: '',
          // role: '',
          // birthday: '',
          // fullName: '',
          // gender: '',
          // phoneNumber: ''
        }
      }
    }
  },
  data() {
    return {
      userInfo: null
    }
  },
  created() {
    console.log(this.$store.getters)
    this.userInfo = Cookies.getJSON('userInfo')
    console.log(Cookies.getJSON('userInfo'))
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scope>
.app-container {
	display: flex;
    justify-content: center;
    align-items: center;
}
.profile-editor-container {
  .chart-wrapper {
    background: #FFFFFF;
    border-radius: 2px;
    h2{
      font-size:18px;
      line-height: 26px;
      color:#212633;
      text-align: center;
      margin: 16px 0px;
    }
    .avatar {
      text-align: center;
      .body {
        img {
          width: 128px;
          margin: auto;
          border-radius: 100%;
        }
      }
    }
    .general-info {
      width: 250px;
      margin: 0 auto 28px;
      .body{
        span{
          font-size: 14px;
          color: #212633;
          line-height: 20px;
        }
        .label {
          color: #525B73;
          font-weight: bold;
          min-width: 100px;
          display: inline-block;
        }
      }
    }
    .btn-center {
      margin: auto;
    }
  }
  .el-dialog__header {
    padding: 0;
    background: #006fe3;
    height: 40px;
    line-height: 40px;
    text-align: center;
    .el-dialog__title {
      color: white;
    }
    .el-icon-close {
      color: white;
      font-size: 22px;
      top: -10px;
      position: relative;
    }
  }
  .el-dialog__body {
    .el-form-item__label {
      padding-bottom: 0;
      font-size: 14px;
      color: #363030;
      font-weight: 500 !important;
    }
  }
  .el-dialog__footer {
    clear: both;
    padding-bottom: 25px;
    .dialog-footer {
      display: flex;
      justify-content: center;
      .btn-cancel {
        height: 32px;
        padding: 0 20px;
        width: 97px;
        background: transparent;
        color: #7C7E81;
        border: 2px solid #7C7E81;
      }
      .btn-cancel:hover {
        background: transparent;
        color: #7C7E81;
        border: 2px solid #7C7E81;
      }
    }
  }
}
</style>
