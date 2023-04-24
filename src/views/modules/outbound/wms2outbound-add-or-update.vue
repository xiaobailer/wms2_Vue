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
      <el-form-item label="出库日期" prop="outboundDate">
        <el-date-picker
          v-model="dataForm.outboundDate"
          value-format="yyyy-MM-dd HH:mm:ss"
          type="datetime"
          placeholder="选择日期时间"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="操作员" prop="operator">
        <el-input v-model="dataForm.operator" placeholder></el-input>
      </el-form-item>
      <el-form-item label="客户" prop="customerId">
        <el-select
          v-model="dataForm.customerId"
          placeholder="请选择"
          v-on:click.native="getCustomer()"
        >
          <el-option
            v-for="item in customers"
            :key="item.customerId"
            :label="item.customerName"
            :value="item.customerId"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      customers:[{
        label:"a",
        value:1
      }],
      dataForm: {
        id: 0,
        outboundDate: "",
        operator: "",
        customerId: "",
        remark: ""
      },
      dataRule: {
        outboundDate: [
          { required: true, message: "不能为空", trigger: "blur" }
        ],
        operator: [{ required: true, message: "不能为空", trigger: "blur" }],
        customerId: [{ required: true, message: "不能为空", trigger: "blur" }],
        remark: [{ required: true, message: "不能为空", trigger: "blur" }]
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
            url: this.$http.adornUrl(
              `/wms/wms2outbound/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.outboundDate = data.wms2Outbound.outboundDate;
              this.dataForm.operator = data.wms2Outbound.operator;
              this.dataForm.customerId = data.wms2Outbound.customerId;
              this.dataForm.remark = data.wms2Outbound.remark;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/wms/wms2outbound/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              outboundDate: this.dataForm.outboundDate,
              operator: this.dataForm.operator,
              customerId: this.dataForm.customerId,
              remark: this.dataForm.remark
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

    //获取客户
    getCustomer() {
      this.$http({
        url: this.$http.adornUrl(
          `/wms/wms2customer/getCustomer`
        ),
        method: "get",
        param: this.dataForm.inboundId
      }).then(({ data }) => {
        this.customers = data.list;
      });
    }
  }
};
</script>
