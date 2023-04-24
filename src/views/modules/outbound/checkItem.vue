<template>
  <div class="mod-config">
    <!-- <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('wms:wms2outbound:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('wms:wms2outbound:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>-->
    <el-button type="primary" @click="getDataList()">所有订单</el-button>
    <el-button type="primary" @click="checkItemIsNo()">未审核</el-button>
    <el-button type="primary" @click="checkItemIsYes()">已审核</el-button>
    <el-button
      type="primary"
      @click="checkItemIsYes()"
      :disabled="dataListSelections.length <= 0"
    >批量审核</el-button>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="productName" header-align="center" align="center" label="商品名称"></el-table-column>
      <el-table-column prop="productSpec" header-align="center" align="center" label="商品规格"></el-table-column>
      <!-- <el-table-column
        prop="productionDate"
        header-align="center"
        align="center"
        label="生产日期">
      </el-table-column>
      <el-table-column
        prop="shelfLife"
        header-align="center"
        align="center"
        label="保质期">
      </el-table-column>-->
      <!-- <el-table-column
        prop="supplierId"
        header-align="center"
        align="center"
        label="供应商">
      </el-table-column>-->
      <el-table-column prop="quantity" header-align="center" align="center" label="数量"></el-table-column>
      <el-table-column prop="price" header-align="center" align="center" label="价格"></el-table-column>
       <el-table-column prop="allPrice" header-align="center" align="center" label="出库总价">
        <template slot-scope="scope">
          <span style="color: red">{{ scope.row.allPrice }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="quantityO" header-align="center" align="center" label="出库数量"></el-table-column>
      <el-table-column prop="priceO" header-align="center" align="center" label="出库价格"></el-table-column>
      <el-table-column prop="allPriceO" header-align="center" align="center" label="出库总价">
        <template slot-scope="scope">
          <span style="color: red">{{ scope.row.allPriceO }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="status" header-align="center" align="center" label="审核状态"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.status === '未审核'" type="text" size="small" @click="confirmAudit(scope.row.id)">确认审核</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "./wms2outbound-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    };
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/wms/wms2outbounddetail/toBeReviewed"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) => {
        console.log(data.page.list);
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
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/wms/wms2outbound/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              }
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
    //查询状态为未审核订单
    checkItemIsNo(){   
        this.$http({
         url: this.$http.adornUrl(`/wms/wms2outbounddetail/getReviewed`),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: "未审核"
        })
      }).then(({ data }) => {
        console.log(data.page.list);
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
    //查询状态已审核订单
    checkItemIsYes(){   
        this.$http({
         url: this.$http.adornUrl(`/wms/wms2outbounddetail/getReviewed`),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: "已审核"
        })
      }).then(({ data }) => {
        console.log(data.page.list);
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
    //确认审核/批量审核
    confirmAudit(id){
     
      var ids = id ? [id]:this.dataListSelections.map(item=>{
        return item.id;
      });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "审核确认" : "批量审核确认"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/wms/wms2outbounddetail/confirmAudit"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
            }.then(()=>{
              this.getDataList();
            }))
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    }
    // //颜色
    // formatAllPrice(row) {
    //   const allPriceO = row.allPriceO;
    //   return `<span style="color: red">${allPriceO}</span>`;
    // }
  }
};
</script>

<style lang="css">
.el-table-column {
  color: red;
}
</style>

