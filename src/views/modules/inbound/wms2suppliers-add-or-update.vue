<template>
  <el-dialog
    :title="!dataForm.supplierId ? '新增' : '修改'"
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
      <el-form-item label="供应商" prop="supplierName">
        <el-input v-model="dataForm.supplierName" placeholder></el-input>
      </el-form-item>
      <el-form-item label="联系人" prop="contact">
        <el-input v-model="dataForm.contact" placeholder></el-input>
      </el-form-item>
      <el-form-item
        label="联系电话"
        prop="phone"
        :rules="[
	      { required: true, message: '请输入手机号', trigger: 'change' },
          { pattern: /^1[3|5|7|8|9]\d{9}$/, message: '请输入正确的号码格式', trigger: 'change' }
	    ]"
      >
        <el-input v-model="dataForm.phone" placeholder="校验"></el-input>
      </el-form-item>
      <el-form-item label="联系地址" prop="address">
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
      detailaddress: "", //详细地址
      citys: regionData,
      organizationName: "", //机构名称
      salesmanUserId: "", //销售
      organizationType: "", //机构类型
      provinceName: "", //省
      cityName: "", //市
      countyName: "", //区
      dataForm: {
        supplierId: 0,
        supplierName: "",
        contact: "",
        phone: "",
        address: ""
      },
      dataRule: {
        supplierName: [
          { required: true, message: "不能为空", trigger: "blur" }
        ],
        contact: [{ required: true, message: "不能为空", trigger: "blur" }],
        phone: [{ required: true, message: "不能为空", trigger: "blur" }],
        address: [{ required: true, message: "不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.supplierId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.supplierId) {
          this.$http({
            url: this.$http.adornUrl(
              `/wms/wms2suppliers/info/${this.dataForm.supplierId}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.supplierName = data.wms2Suppliers.supplierName;
              this.dataForm.contact = data.wms2Suppliers.contact;
              this.dataForm.phone = data.wms2Suppliers.phone;
              this.dataForm.address = data.wms2Suppliers.address;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.dataForm.address = this.dataForm.address + this.detailaddress
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/wms/wms2suppliers/${
                !this.dataForm.supplierId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              supplierId: this.dataForm.supplierId || undefined,
              supplierName: this.dataForm.supplierName,
              contact: this.dataForm.contact,
              phone: this.dataForm.phone,
              address: this.dataForm.address
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
        this.provinceName +
        this.cityName +
        this.countyName
        console.log(this.dataForm.address)
    }
  }
};
</script>
