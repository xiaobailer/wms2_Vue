<template>
  <el-form ref="form" :model="dataForm" label-width="80px">
    <el-form-item label="出库单" prop="outboundId">
      <el-select
        v-model="dataForm.outboundId"
        placeholder="请选择"
        v-on:click.native="getAllOutBoundId()"
      >
        <el-option v-for="item in outboundIds" :key="item.id" :label="item.id" :value="item.id"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="出库产品" prop="outProduct">
      <el-select
        v-model="dataForm.outProduct"
        placeholder="请选择"
        v-on:click.native="getShouldOutPro()"
      >
        <el-option
          v-for="item in outProducts"
          :key="item.productId"
          :label="item.productName"
          :value="item.productId"
        ></el-option>
      </el-select>
    </el-form-item>
    <!-- <el-form-item label="" prop="productId">
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
    </el-form-item>-->
    <el-form-item label="产品数量" prop="quantity">
      <el-input v-model="dataForm.quantity" placeholder="获取小于或等于该数量"  v-on:click.native="getHouseId()"></el-input>
    </el-form-item>
    <el-form-item label="产品单价" prop="price">
      <el-input v-model="dataForm.price" placeholder="数量"></el-input>
    </el-form-item>
    <!-- <el-form-item label="状态" prop="status">
      <el-select v-model="dataForm.status" placeholder="请选择"
      v-on:click.native="getHouseId()">
        <el-option v-for="item in status" :key="item.value" :label="item.label" :value="item.value"></el-option>
      </el-select>
    </el-form-item> -->
    <el-form-item>
      <el-button type="primary" @click="dataFormSubmit()">立即创建</el-button>
      <el-button>取消</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  activated() {
    this.getShouldOutPro();
  },
  data() {
    return {
      status: "未出库",
      outboundIds: [
        {
          label: "a",
          value: 1
        }
      ],
      outProducts: [
        {
          label: "a",
          value: 1
        }
      ],
      visible: false,
      warehouseId: "",
      dataForm: {
        id: 0,
        outboundId: "",
        outProduct: "",
        quantity: "",
        price: "",
        status: "",
        warehouseId: ""
      },
      dataRule: {
        outboundId: [{ required: true, message: "不能为空", trigger: "blur" }],
        outProduct: [{ required: true, message: "不能为空", trigger: "blur" }],
        quantity: [{ required: true, message: "不能为空", trigger: "blur" }],
        price: [{ required: true, message: "不能为空", trigger: "blur" }],
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
            url: this.$http.adornUrl(
              `/wms/wms2outbounddetail/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.outboundId = data.wms2OutboundDetail.outboundId;
              this.dataForm.productId = data.wms2OutboundDetail.productId;
              this.dataForm.quantity = data.wms2OutboundDetail.quantity;
              this.dataForm.price = data.wms2OutboundDetail.price;
              this.dataForm.status = data.wms2OutboundDetail.status;
              this.dataForm.warehouseId = this.warehouseId;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
          this.dataForm.warehouseId = this.warehouseId
           this.dataForm.status = this.status
          //发送请求获取该产品id的仓库
          this.$http({
            url: this.$http.adornUrl(
              `/wms/wms2outbounddetail/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              outboundId: this.dataForm.outboundId,
              productId: this.dataForm.outProduct,
              quantity: this.dataForm.quantity,
              price: this.dataForm.price,
              warehouseId: this.dataForm.warehouseId,
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
           // location.reload();
          });
    },
    //获取状态为待出库产品，没有进行过出仓的产品
    getShouldOutPro() {
     this.$http({
       //待修改这个请求
          url: this.$http.adornUrl('/wms/wms2outbound/toBeOutProduct'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({ data }) => {
        console.log(data.page);
        const products = data.page.list.map(item=>({
          productId: item.productId,
          productName: item.productName
        }))
        this.outProducts = products;
      });
    },
    //获取入库单
    getAllOutBoundId() {
      this.$http({
        url: this.$http.adornUrl(`/wms/wms2outbound/getAllOutBoundId`),
        method: "get"
      }).then(({ data }) => {
        console.log(data.list);
        this.outboundIds = data.list;
      });
    },
    getHouseId(){
       this.$http(
            { 
              url: this.$http.adornUrl(`/wms/wms2outbounddetail/getProductById/${this.dataForm.outProduct}`),
              method: "get",
              param: this.dataForm.outProduct,
            }
          ).then(({data})=>{
            console.log(data)
            this.warehouseId = data.houseId
          })
          
    }
  }
};
</script>
