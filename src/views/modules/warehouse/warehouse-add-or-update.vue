<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="仓库名" prop="name">
        <el-input v-model="dataForm.name" placeholder></el-input>
      </el-form-item>
      <el-form-item label="仓库地址" prop="address">
        <div>
          <el-cascader
            placeholder="可搜索"
            :options="citys"
            filterable
            clearable
            @change="handleChange"
          >
            <template slot-scope="{ node, data }">
              <span>{{ data.label }}</span>
              <span v-if="!node.isLeaf">({{ data.children.length }})</span>
            </template>
          </el-cascader>
          <el-input placeholder="请输入详细地址" style="width: 50%" v-model="detailaddress"></el-input>
        </div>
      </el-form-item>
      <el-form-item label="管理员" prop="manager">
        <el-input v-model="dataForm.manager" placeholder></el-input>
      </el-form-item>
      <el-form-item label="总容量" prop="capacity">
        <el-input v-model="dataForm.capacity" placeholder></el-input>
      </el-form-item>
      <el-form-item label="占用量" prop="occupied">
        <el-input v-model="dataForm.occupied" placeholder></el-input>
      </el-form-item>
      <el-form-item label="仓库状态" prop="status">

        <el-select v-model="dataForm.status" placeholder="请选择">
          <el-option
            v-for="item in statusItem"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import {
  provinceAndCityData,
  regionData,
  provinceAndCityDataPlus,
  regionDataPlus,
  CodeToText,
  TextToCode
} from "element-china-area-data";
export default {
  data() {
    return {
      visible: false,
      statusItem: [
        {
          label: "可用",
          value: "可用"
        },
        {
          label: "不可用",
          value: "不可用"
        }
      ],
      detailaddress: "", //详细地址
      citys: regionData,
      organizationName: "", //机构名称
      salesmanUserId: "", //销售
      organizationType: "", //机构类型
      provinceName: "", //省
      cityName: "", //市
      countyName: "", //区
      dataForm: {
        id: 0,
        name: "",
        address: "",
        manager: "",
        capacity: "",
        occupied: "",
        status: ""
      },
      dataRule: {
        name: [{ required: true, message: "不能为空", trigger: "blur" }],
        address: [{ required: true, message: "不能为空", trigger: "blur" }],
        manager: [{ required: true, message: "不能为空", trigger: "blur" }],
        capacity: [{ required: true, message: "不能为空", trigger: "blur" }],
        occupied: [{ required: true, message: "不能为空", trigger: "blur" }],
        status: [{ required: true, message: "不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/wms/warehouse/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.warehouse.name;
              this.dataForm.address = data.warehouse.address;
              this.dataForm.manager = data.warehouse.manager;
              this.dataForm.capacity = data.warehouse.capacity;
              this.dataForm.occupied = data.warehouse.occupied;
              this.dataForm.status = data.warehouse.status;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.dataForm.address = this.dataForm.address + this.detailaddress;
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/wms/warehouse/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              address: this.dataForm.address,
              manager: this.dataForm.manager,
              capacity: this.dataForm.capacity,
              occupied: this.dataForm.occupied,
              status: this.dataForm.status
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
    //地址
    handleChange(value) {
      let cityNames = [];
      value.forEach(e => {
        cityNames.push(CodeToText[e]);
      });
      this.provinceName = cityNames[0];
      this.cityName = cityNames[1];
      this.countyName = cityNames[2];
      console.log(this.provinceName, this.cityName, this.countyName);
      this.dataForm.address =
        this.provinceName + this.cityName + this.countyName;
      console.log(this.dataForm.address);
    }
  }
};
</script>
