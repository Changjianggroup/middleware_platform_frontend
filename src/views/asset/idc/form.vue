<template>
  <div>
    <el-form ref="form" :model="form" :rules="rules" label-width="100px" class="idc-form">
      <el-form-item label="机房名称" prop="name">
        <el-input v-model="form.name" placeholder="请输入机房名称"></el-input>
      </el-form-item>
      <el-form-item label="机房地址" prop="address">
        <el-input v-model="form.address" placeholder="请输入机房地址"></el-input>
      </el-form-item>
      <el-form-item label="机房电话" prop="phone">
        <el-input v-model="form.phone" placeholder="请输入机房电话"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="form.remark" placeholder="请输入备注"></el-input>
      </el-form-item>
      <el-form-item class="button-right">
        <el-button size="small" @click="cancelForm">取消</el-button>
        <el-button size="small" type="primary" @click="submitForm">{{ value }}</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'IdcForm',
  props: {
    form: {
      type: Object,
      default: function() {
        return {
          name: '',
          address: '',
          phone: '',
          remark: ''
        }
      }
    },
    value: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      rules: {
        name: [
          { required: true, message: '请输入机房名称', trigger: 'blur' },
          { max: 16, message: '长度不能超过16个字符', trigger: 'blur' }
        ],
        address: [
          { max: 128, message: '长度不能超过128个字符', trigger: 'blur' }
        ],
        phone: [
          { max: 16, message: '长度不能超过16个字符', trigger: 'blur' }
        ],
        remark: [
          { max: 255, message: '长度不能超过255个字符', trigger: 'blur' }

        ]
      }
    }
  },
  methods: {
    submitForm() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          this.$emit('submit', this.form)
        } else {
          return
        }
      })
    },
    cancelForm() {
      this.$emit('cancel')
    }
  }
}
</script>

<style lang="scss" scoped>
.idc-form {
  position: relative;
  display: block
  }
    .button-right {
      text-align: right;
  }
</style>
