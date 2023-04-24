<template>
  <el-dialog
    :title="!dataForm.inboundId ? '新增' : '修改'"
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
      <el-form-item label="入库时间" prop="inboundTime">
        <div class="block">
          <el-date-picker v-model="dataForm.inboundTime" type="date" placeholder="选择日期"></el-date-picker>
        </div>
      </el-form-item>
      <el-form-item label="入库员" prop="inboundUser">
        <el-input v-model="dataForm.inboundUser" placeholder></el-input>
      </el-form-item>
      <el-form-item label="供应商" prop="supplierId">
          <el-select v-model="dataForm.supplierId" placeholder="请选择" v-on:click.native="getSupplier()">
        <el-option
          v-for="item in repoeds"
          :key="item.supplierId"
          :label="item.supplierName"
          :value="item.supplierId"
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
       repoeds: [
        {
          label: "a",
          value: 1
        }
      ],
      visible: false,
        pickerOptions: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
        shortcuts: [
          {
            text: "今天",
            onClick(picker) {
              picker.$emit("pick", new Date());
            }
          },
          {
            text: "昨天",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24);
              picker.$emit("pick", date);
            }
          },
          {
            text: "一周前",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", date);
            }
          }
        ]
      },
      dataForm: {
        inboundId: 0,
        inboundTime: "",
        inboundUser: "",
        supplierId: "",
        remark: ""
      },
      dataRule: {
        inboundTime: [{ required: true, message: "不能为空", trigger: "blur" }],
        inboundUser: [{ required: true, message: "不能为空", trigger: "blur" }],
        supplierId: [{ required: true, message: "不能为空", trigger: "blur" }],
        remark: [{ required: true, message: "不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.inboundId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.inboundId) {
          this.$http({
            url: this.$http.adornUrl(
              `/wms/wms2inbounds/info/${this.dataForm.inboundId}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.inboundTime = data.wms2Inbounds.inboundTime;
              this.dataForm.inboundUser = data.wms2Inbounds.inboundUser;
              this.dataForm.supplierId = data.wms2Inbounds.supplierId;
              this.dataForm.remark = data.wms2Inbounds.remark;
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
              `/wms/wms2inbounds/${
                !this.dataForm.inboundId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              inboundId: this.dataForm.inboundId || undefined,
              inboundTime: this.dataForm.inboundTime,
              inboundUser: this.dataForm.inboundUser,
              supplierId: this.dataForm.supplierId,
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
      //获取供应商
    getSupplier() {
      this.$http({
        url: this.$http.adornUrl(`/wms/wms2suppliers/getSupplier`),
        method: "get"
      }).then(({ data }) => {
        this.repoeds = data.list;
      });
    },
  }
};
</script>
