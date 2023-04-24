<template>
  <el-dialog
    :title="!dataForm.productId ? '新增' : '修改'"
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
      <el-form-item label="商品名称" prop="productName">
        <el-input v-model="dataForm.productName" placeholder></el-input>
      </el-form-item>
      <el-form-item label="商品规格" prop="productSpec">
        <el-input v-model="dataForm.productSpec" placeholder></el-input>
      </el-form-item>
      <el-form-item label="生产日期" prop="productionDate">
        <div class="block">
          <el-date-picker v-model="dataForm.productionDate" type="date" placeholder="选择日期"></el-date-picker>
        </div>
      </el-form-item>
      <el-form-item label="保质期" prop="shelfLife">
        <el-input v-model="dataForm.shelfLife" placeholder></el-input>
      </el-form-item>
      <el-form-item label="供应商" prop="supplierId">
        <el-select
          v-model="dataForm.supplierId"
          placeholder="请选择"
          v-on:click.native="getSupplier()"
        >
          <el-option
            v-for="item in suppliers"
            :key="item.supplierId"
            :label="item.supplierName"
            :value="item.supplierId"
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
export default {
  data() {
    return {
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
      suppliers: [
        {
          label: "a",
          value: 0
        }
      ],
      visible: false,
      dataForm: {
        productId: 0,
        productName: "",
        productSpec: "",
        productionDate: "",
        shelfLife: "",
        supplierId: ""
      },
      dataRule: {
        productName: [{ required: true, message: "不能为空", trigger: "blur" }],
        productSpec: [{ required: true, message: "不能为空", trigger: "blur" }],
        productionDate: [
          { required: true, message: "不能为空", trigger: "blur" }
        ],
        shelfLife: [{ required: true, message: "不能为空", trigger: "blur" }],
        supplierId: [{ required: true, message: "不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.productId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.productId) {
          this.$http({
            url: this.$http.adornUrl(
              `/wms/wms2products/info/${this.dataForm.productId}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.productName = data.wms2Products.productName;
              this.dataForm.productSpec = data.wms2Products.productSpec;
              this.dataForm.productionDate = data.wms2Products.productionDate;
              this.dataForm.shelfLife = data.wms2Products.shelfLife;
              this.dataForm.supplierId = data.wms2Products.supplierId;
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
              `/wms/wms2products/${
                !this.dataForm.productId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              productId: this.dataForm.productId || undefined,
              productName: this.dataForm.productName,
              productSpec: this.dataForm.productSpec,
              productionDate: this.dataForm.productionDate,
              shelfLife: this.dataForm.shelfLife,
              supplierId: this.dataForm.supplierId
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
    //获取供应商id
    //获取供应商
    getSupplier() {
      this.$http({
        url: this.$http.adornUrl(`/wms/wms2suppliers/getSupplier`),
        method: "get"
      }).then(({ data }) => {
        // this.repoeds = data.list;
        const suppliers = data.list.map(item => ({
          supplierId: item.supplierId,
          supplierName: item.supplierName
        }));
        this.suppliers = suppliers;
      });
    }
  }
};
</script>
