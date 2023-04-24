<template>
  <el-dialog
    :title="!dataForm.inboundId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="productId">
      <el-input v-model="dataForm.productId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="quantity">
      <el-input v-model="dataForm.quantity" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="price">
      <el-input v-model="dataForm.price" placeholder=""></el-input>
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
    data () {
      return {
        visible: false,
        dataForm: {
          inboundId: 0,
          productId: '',
          quantity: '',
          price: ''
        },
        dataRule: {
          productId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          quantity: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          price: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.inboundId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.inboundId) {
            this.$http({
              url: this.$http.adornUrl(`/wms/wms2inbounddetails/info/${this.dataForm.inboundId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.productId = data.wms2InboundDetails.productId
                this.dataForm.quantity = data.wms2InboundDetails.quantity
                this.dataForm.price = data.wms2InboundDetails.price
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/wms/wms2inbounddetails/${!this.dataForm.inboundId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'inboundId': this.dataForm.inboundId || undefined,
                'productId': this.dataForm.productId,
                'quantity': this.dataForm.quantity,
                'price': this.dataForm.price
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
