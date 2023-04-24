
<template>
  <div class="container">
    <el-card class="card">
      <div
        slot="header"
      >已入库产品表&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;空闲仓位</div>
      <el-button
        type="primary"
        @click="handleButtonClick()"
        :disabled="dataListProductionSelections.length <= 0 || dataListHouseSelections.length <=0" 
      >立即分配</el-button>
      <el-row>
        <el-col :span="12">
          <el-table
            :data="dataList"
            border
            v-loading="dataListLoading"
            style="width: 100%"
            @selection-change="selectionProductChangeHandle"
          >
            <!-- table columns -->
            <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
            <el-table-column prop="productName" header-align="center" align="center" label="商品名称"></el-table-column>
            <el-table-column prop="productSpec" header-align="center" align="center" label="商品规格"></el-table-column>
            <el-table-column
              prop="productionDate"
              header-align="center"
              align="center"
              label="生产日期"
            ></el-table-column>
            <el-table-column prop="shelfLife" header-align="center" align="center" label="保质期"></el-table-column>
            <!-- <el-table-column
        prop="supplierId"
        header-align="center"
        align="center"
        label="供应商">
            </el-table-column>-->
            <el-table-column prop="quantity" header-align="center" align="center" label="数量"></el-table-column>
          </el-table>
        </el-col>
        <el-col :span="12">
          <el-table
            border
            :data="dataHouseList"
            style="width: 100%"
            @selection-change="handleRowClick"
          >
            <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
            <!-- <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="仓库编号">
            </el-table-column>-->
            <el-table-column prop="name" header-align="center" align="center" label="仓库名称"></el-table-column>
            <el-table-column prop="address" header-align="center" align="center" label="仓库地址"></el-table-column>
            <el-table-column prop="manager" header-align="center" align="center" label="仓库管理员"></el-table-column>
            <!-- <el-table-column
        prop="capacity"
        header-align="center"
        align="center"
        label="仓库容量">
      </el-table-column>
      <el-table-column
        prop="occupied"
        header-align="center"
        align="center"
        label="已占用容量">
            </el-table-column>-->
            <el-table-column prop="status" header-align="center" align="center" label="仓库状态"></el-table-column>
          </el-table>
        </el-col>
      </el-row>
    </el-card>
  </div>
</template>



<script>
export default {
  activated() {
    this.getHaveInProduct();
    this.getWellHouse();
  },
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [],
      selectHouse: "",
      dataHouseList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListProductionSelections: [

      ],
      dataListHouseSelections: [],
      addOrUpdateVisible: false
    };
  },
  methods: {
    getHaveInProduct() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/wms/wms2products/listInProduct"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    getWellHouse() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/wms/warehouse/getWellHouse"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataHouseList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataHouseList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // // 处理行点击事件
    // handleRowClick(row) {
    //   // 如果是商品行，显示商品详情弹窗
    //   if (this.dataList.includes(row)) {
    //     this.selectedGoods = row;
    //     //拿到选行id
    //     console.log(this.selectedGoods.productId);
    //     console.log(this.selectedGoods);
    //     this.goodsDialogVisible = true;
    //   }
    //   // 如果是仓库行，进行仓库分配操作
    //   else if (this.dataHouseList.includes(row)) {
    //     this.assignWarehouse(row);
    //   }
    // },
    // 产品多选
    selectionProductChangeHandle(val) {
      this.dataListProductionSelections = val;
      console.log(val);
    },
    //仓库单选
    handleRowClick(val) {
      if (val.length <= 1) {
        this.dataListHouseSelections = val;
        console.log(this.dataListHouseSelections);
      } else {
        this.$message({
          message: "警告,只能选择一个仓库",
          type: "warning"
        });
      }
    },
    //前端分配请求
    handleButtonClick(id1, id2) {
      var psid = id1
        ? [id1]
        : this.dataListProductionSelections.map(item => {
            return item.productId;
          });
      console.log(psid);

      var hsid =id2 ? [id2] : this.dataListHouseSelections.map(item1 => {
        return item1.id;
      });

      var requestData = {   
          productids:psid,
          houseids:hsid
      }
      console.log(hsid);
       this.$confirm(`确定对[id=${psid.join(',')}]进行[${psid ? '分配' : '批量分配'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(()=>{
          this.$http({
        //创建仓库与商品的关联表
        url: this.$http.adornUrl("/wms/wms2products/addInHouseProudct"),
        method: "post",
        data: requestData,
      }).then(({data})=>{
        if (data && data.code === 0) {
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 1500,
            onClose: () => {
             this.getHaveInProduct();
            }
          });
        } else {
          this.$message.error(data.msg);
        }
      })
        })
    }
  }
};
</script>

<style>
.container {
  margin: 20px;
}

.card {
  width: 100%;
  margin-bottom: 20px;
}
</style>