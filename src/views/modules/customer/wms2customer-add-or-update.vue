<template>
  <el-dialog
    :title="!dataForm.customerId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="公司名" prop="customerName">
      <el-input v-model="dataForm.customerName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="客户类型" prop="customerType">
      <el-input v-model="dataForm.customerType" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="联系人" prop="contactPerson">
      <el-input v-model="dataForm.contactPerson" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="联系电话" prop="contactPhone">
      <el-input v-model="dataForm.contactPhone" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="联系地址" prop="address">
      <el-input v-model="dataForm.address" placeholder=""></el-input>
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
          customerId: 0,
          customerName: '',
          customerType: '',
          contactPerson: '',
          contactPhone: '',
          address: ''
        },
        dataRule: {
          customerName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          customerType: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          contactPerson: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          contactPhone: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          address: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.customerId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.customerId) {
            this.$http({
              url: this.$http.adornUrl(`/wms/wms2customer/info/${this.dataForm.customerId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.customerName = data.wms2Customer.customerName
                this.dataForm.customerType = data.wms2Customer.customerType
                this.dataForm.contactPerson = data.wms2Customer.contactPerson
                this.dataForm.contactPhone = data.wms2Customer.contactPhone
                this.dataForm.address = data.wms2Customer.address
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
              url: this.$http.adornUrl(`/wms/wms2customer/${!this.dataForm.customerId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'customerId': this.dataForm.customerId || undefined,
                'customerName': this.dataForm.customerName,
                'customerType': this.dataForm.customerType,
                'contactPerson': this.dataForm.contactPerson,
                'contactPhone': this.dataForm.contactPhone,
                'address': this.dataForm.address
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
