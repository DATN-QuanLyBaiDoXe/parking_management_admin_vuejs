<template>
  <div class="body">
    <div>
      <el-input
        class="input-search"
        placeholder="Tìm kiếm"
        prefix-icon="el-icon-search"
        v-model="queryPage.search"
        @change="onChangeInputSearch"
      >
      </el-input>
    </div>
    <div class="mb-4">
		<div id="vehicle-table">
      <!-- <div class="table-responsive"> -->
      <div v-if="multiSelected.length > 0" id="select-all-delete">
        <i class="el-icon-close" @click="closeDelete()" />
        <span v-if="multiSelected.length < 10">0{{ multiSelected.length }}</span
        ><span v-else>{{ multipleSelection.length }}</span> mục được chọn
        <el-button
          type="danger"
          :loading="loading_delete_all"
          @click="destroyMulti()"
          ><i class="el-icon-delete" /> Xóa</el-button
        >
      </div>
      <el-table
        ref="multipleTable"
        v-loading="loading"
        :data="vehicleList"
        style="width: 100%"
        @selection-change="handleSelectionChange"
        @row-click="rowClicked"
      >
        <el-table-column type="selection" width="70"> </el-table-column>
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
        <el-table-column prop="place" label="Biển số"> </el-table-column>
        <el-table-column prop="color" label="Màu sắc"> </el-table-column>
        <el-table-column prop="brand" label="Thương hiệu"> </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="queryPage.page"
        :page-sizes="[1, 5, 10, 15, 20]"
        :page-size="queryPage.size"
        layout="total, sizes, prev, pager, next, jumper"
        :total="queryPage.total"
      >
      </el-pagination>
	</div>
      <div id="detail-vehicle" class="violation-ruleForm" style="width: 0">
        <i class="el-icon-close" @click="closeDetail()" />
        <template v-if="vehicleDetail">
          <div class="avatar" />

          <template v-if="vehicleDetail.vehicleType">
            <div class="item-label">Loại phương tiện</div>
            <div class="item-text">{{ vehicleDetail.vehicleType | getVehicleTypeName }}</div>
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
            <el-button type="info" @click="handleEdit(vehicleDetail)"
              ><i class="el-icon-edit" /> Sửa
            </el-button>
            <el-button
              type="danger"
			  style="float: right;"
              :loading="loadingVehicle"
              @click="destroy(vehicleDetail)"
              ><i class="el-icon-delete" /> Xóa
            </el-button>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";

export default {
  name: "vehicles",
  data() {
    return {
      loading: true,
	  loadingVehicle: false,
      vehicleList: [],
      multiSelected: [],
      allSelected: false,
      loading_delete_all: false,
      queryPage: {
        page: 0,
        size: 10,
        total: 0,
        search: "",
      },
      vehicle: {
        uuid: "",
        vehicleType: "",
        place: "",
        color: "",
        brand: "",
        ownerName: "",
        status: null,
      },
	  vehicleDetail: {},
    };
  },
  created() {
    this.getVehicle();
  },
  filters: {
    getVehicleTypeName: function (value) {
      let type = "";
      if (!value) type = "";
      if (value === "CAR") type = "Ô tô";
      if (value === "MOTO") type = "Xe máy";
      if (value === "TRAM") type = "Xe đạp điện";
      if (value === "BIKE") type = "Xe đạp";
      return type;
    },
  },
  methods: {
    closeDialog(formName) {
      this.$refs[formName].clearValidate();
      this.$refs[formName].resetFields();
    },
    toggleAll(checked) {
      this.multiSelected = checked ? this.userList.slice() : [];
    },
    handleSizeChange(size) {
      this.queryPage.size = size;
      this.getVehicle();
    },
    handleCurrentChange(page) {
      this.queryPage.page = page;
      this.getVehicle();
    },
    onChangeInputSearch() {
      this.queryPage.page = 0;
      this.getVehicle();
    },
    handleSelectionChange(val) {
      this.multiSelected = val;
    },
    getVehicle() {
		this.loading = true;
      const headers = {
        "Content-Type": "multipart/form-data",
        Authorization: "Bearer " + Cookies.get("access-token"),
      };
      axios
        .get(process.env.VUE_APP_API + "vehicle", {
          headers: headers,
          params: {
            page: this.queryPage.page > 0 ? this.queryPage.page - 1 : 0,
            size: this.queryPage.size,
            search: this.queryPage.search,
          },
        })
        .then((res) => {
          if (res.data) {
            this.vehicleList = res.data.data;
            this.queryPage.total = res.data.total;
			this.loading = false;
          }
        })
        .catch((err) => {
          console.log(err);
          this.loading = false;
          this.$message({
            type: "error",
            message: "Có lỗi xảy ra vui lòng thử lại sau!",
          });
        });
    },

    closeDelete() {
      this.$refs.multipleTable.clearSelection();
      this.multipleSelection = [];
    },

    destroyMulti() {
      this.$confirm("Bạn có chắc chắn muốn xóa lựa chọn này?", "", {
        confirmButtonText: "Xác nhận",
        cancelButtonText: "Thoát",
        type: "warning",
      })
        .then(() => {
          const headers = {
            "Content-Type": "application/json",
            Authorization: "Bearer " + Cookies.get("access-token"),
          };
          this.loading_delete_all = true;
          let params = this.multiSelected.map(function (val) {
            return val.uuid;
          });
          params = {
            uuids: params,
          };
          axios
            .delete(process.env.VUE_APP_API + "vehicle", {
              headers: headers,
              data: params,
            })
            .then((res) => {
              this.multiSelected = [];
              this.onChangeInputSearch();
              this.loading_delete_all = false;
            })
            .catch((err) => {
              this.multiSelected = [];
              this.onChangeInputSearch();
              this.loading_delete_all = false;
              this.$message({
                type: "warning",
                message: "Có lỗi xảy ra vui lòng thử lại",
              });
            });
        })
        .catch(() => {});
    },

    async rowClicked(row) {
      this.flag_active = row.uuid;
      if (row) {
        this.vehicleDetail = row;
        if (this.vehicleDetail) {
          document.getElementById("vehicle-table").style.width =
            "calc(100% - 500px)";
          setTimeout(function () {
            document.getElementById("detail-vehicle").style.display = "block";
            document.getElementById("detail-vehicle").style.width = "420px";
          }, 500);
        }
      } else {
        this.$message({
          message: "Không tìm thấy phương tiện",
          type: "error",
        });
        this.onChangeInputSearch();
      }
    },
	
    closeDetail() {
      document.getElementById('detail-vehicle').style.display = 'none'
      document.getElementById('detail-vehicle').style.width = '0'
      document.getElementById('vehicle-table').style.width = 'calc(100%)'
    },

	destroy(vehicle) {
      this.$confirm("Bạn có chắc chắn muốn xóa lựa chọn này?", "", {
        confirmButtonText: "Xác nhận",
        cancelButtonText: "Thoát",
        type: "warning",
      })
        .then(() => {
          const headers = {
            "Content-Type": "application/json",
            Authorization: "Bearer " + Cookies.get("access-token"),
          };
          this.loadingVehicle = true;
		  let params = this.multiSelected.map(function (val) {
            return val.uuid;
          });
		  params.push(vehicle.uuid)
          params = {
            uuids: params,
          };
          axios
            .delete(process.env.VUE_APP_API + "vehicle", {
              headers: headers,
              data: params,
            })
            .then((res) => {
              this.multiSelected = [];
              this.onChangeInputSearch();
			  this.closeDetail()
              this.loadingVehicle = false;
            })
            .catch((err) => {
              this.multiSelected = [];
              this.onChangeInputSearch();
              this.loadingVehicle = false;
              this.$message({
                type: "warning",
                message: "Có lỗi xảy ra vui lòng thử lại",
              });
            });
        })
        .catch(() => {});
    },
  },
  watch: {
    multiSelected(newValue) {
      if (newValue.length === 0) {
        this.indeterminate = false;
        this.allSelected = false;
      } else if (newValue.length === this.userList.length) {
        this.indeterminate = false;
        this.allSelected = true;
      } else {
        this.indeterminate = true;
        this.allSelected = false;
      }
    },
  },
};
</script>

<style>
</style>