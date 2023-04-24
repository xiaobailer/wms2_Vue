<template>
  <el-form ref="form" :model="dataForm" label-width="80px">
    <el-form-item label="入库单" prop="inboundId">
      <el-select v-model="dataForm.inboundId" placeholder="请选择" v-on:click.native="getInboundId()">
        <el-option
          v-for="item in inboundIds"
          :key="item.inboundId"
          :label="item.label"
          :value="item.inboundId"
        ></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="供应商" prop="supplierId">
      <el-select
        v-model="dataForm.supplierId"
        placeholder="请选择"
        v-on:click.native="getSupplierByInId()"
      >
        <el-option
          v-for="item in supplierIds"
          :key="item.supplierId"
          :label="item.supplierName"
          :value="item.supplierId"
        ></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="待入库" prop="productId">
      <el-select
        v-model="dataForm.productId"
        placeholder="请选择"
        v-on:click.native="getSupplierProductNoHouse()"
      >
        <el-option
          v-for="item in products"
          :key="item.productId"
          :label="item.productName"
          :value="item.productId"
        ></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="产品数量" prop="quantity">
      <el-input v-model="dataForm.quantity" placeholder="下拉选择产品"></el-input>
    </el-form-item>
    <el-form-item label="产品单价" prop="price">
      <el-input v-model="dataForm.price" placeholder="下拉选择产品"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="dataFormSubmit()">立即创建</el-button>
      <el-button>取消</el-button>
    </el-form-item>
  </el-form>
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
      inboundIds: [
        {
          label: "",
          value: 1
        }
      ],
      repoeds: [
        {
          label: "a",
          value: 1
        }
      ],
      supplierIds: [{ label: "a", value: 1 }],
      products: [
        {
          label: "a",
          value: 1
        }
      ],
      visible: false,
      dataForm: {
        inboundTime: "",
        inboundUser: "",
        inboundId: "",
        productId: "",
        quantity: "",
        price: "",
        remark: "",
        supplierId: ""
      },
      dataRule: {
        productId: [{ required: true, message: "不能为空", trigger: "blur" }],
        quantity: [{ required: true, message: "不能为空", trigger: "blur" }],
        price: [{ required: true, message: "不能为空", trigger: "blur" }]
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
              `/wms/wms2inbounddetails/info/${this.dataForm.inboundId}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.productId = data.wms2InboundDetails.productId;
              this.dataForm.quantity = data.wms2InboundDetails.quantity;
              this.dataForm.price = data.wms2InboundDetails.price;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$http({
        url: this.$http.adornUrl(`/wms/wms2inbounddetails/saveInboundDetails`),
        method: "post",
        data: this.$http.adornData({
          inboundId: this.dataForm.inboundId,
          productId: this.dataForm.productId,
          quantity: this.dataForm.quantity,
          price: this.dataForm.price
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 1500
          });
          // 提交完成后清空页面
          this.dataForm = {}
        } else {
          this.$message.error(data.msg);
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
    //获取供应商中还未入库产品
    getSupplierProductNoHouse() {
      if (this.dataForm.supplierId) {
        let param = this.dataForm.supplierId;
        console.log(param);
        this.$http({
          url: this.$http.adornUrl(
            `/wms/wms2products/getSupplierProductNoHouse/${
              this.dataForm.supplierId
            }`
          ),
          method: "get",
          param: param
        }).then(({ data }) => {
          this.products = data.list;
        });
      }
    },

    //获取入库号
    getInboundId() {
      this.$http({
        url: this.$http.adornUrl(`/wms/wms2inbounds/getInboundId`),
        method: "get"
      }).then(({ data }) => {
        this.inboundIds = data.list;
        console.log(this.inboundIds);
      });
    },
    //根据入库号获取供应商
    getSupplierByInId() {
      this.$http({
        url: this.$http.adornUrl(
          `/wms/wms2inbounds/getSupplierByInboundId/${this.dataForm.inboundId}`
        ),
        method: "get",
        param: this.dataForm.inboundId
      }).then(({ data }) => {
        this.supplierIds = data.list;
      });
    },
  }
};
</script>
