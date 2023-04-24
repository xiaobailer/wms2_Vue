<template>
  <el-form ref="form" :model="form" label-width="80px">
    <el-form-item label="供应商">
      <el-input v-model="form.supplierName"></el-input>
    </el-form-item>
    <el-form-item label="联系人">
      <el-input v-model="form.contact"></el-input>
    </el-form-item>
    <el-form-item label="联系电话">
      <el-input v-model="form.phone"></el-input>
    </el-form-item>
     <el-form-item label="联系地址" prop="address">
      <el-input v-model="form.address" placeholder="细节待优化"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">立即创建</el-button>
      <el-button>取消</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
  export default {
    data() {
      return {
        form: {
          supplierName: '',
          contact: '',
          phone: '',
          address: '',
        }
      }
    },
    methods: {
      onSubmit() {
        console.log('submit!');
      }
    }
  }
</script>
