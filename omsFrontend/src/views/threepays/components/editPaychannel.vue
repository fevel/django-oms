<template>
  <el-form :model="rowdata" :rules="rules" ref="ruleForm" label-width="100px">
    <el-form-item label="平台" prop="platform">
      <el-input v-model="rowdata.platform" disabled></el-input>
    </el-form-item>
    <el-form-item label="商户" prop="merchant">
      <el-input v-model="rowdata.merchant" disabled></el-input>
    </el-form-item>
    <el-form-item label="类型" prop="type">
      <el-input v-model="rowdata.type" disabled></el-input>
    </el-form-item>
    <el-form-item label="费率" prop="rate">
      <el-input v-model="rowdata.rate"></el-input>
    </el-form-item>
    <el-form-item label="key信息" prop="keyinfo">
      <el-input v-model="rowdata.keyinfo" type="textarea" :autosize="{ minRows: 5, maxRows: 10}"></el-input>
    </el-form-item>
    <el-form-item label="转发域名" prop="m_forwardurl">
      <el-input v-model="rowdata.m_forwardurl"></el-input>
    </el-form-item>
    <el-form-item label="提交域名" prop="m_submiturl">
      <el-input v-model="rowdata.m_submiturl"></el-input>
    </el-form-item>
    <el-form-item label="通知人" prop="action_user">
      <el-select v-model="rowdata.action_user" filterable placeholder="请选择通知人">
        <el-option v-for="item in users" :key="item.id" :value="item.username"></el-option>
      </el-select>
      <el-checkbox v-model="sendnotice">发送通知</el-checkbox>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm('ruleForm')">更新</el-button>
    </el-form-item>
  </el-form>
</template>
<script>
import { putPayChannel } from 'api/threeticket'
import { getUser } from 'api/user'
import { postSendmessage } from 'api/tool'

export default {
  props: ['rowdata'],
  data() {
    return {
      create_user: localStorage.getItem('username'),
      rules: {
        type: [
          { required: true, message: '请输入正确的内容', trigger: 'change' }
        ],
        m_id: [
          { required: true, message: '请输入正确的内容', trigger: 'blur' }
        ],
        action_user: [
          { required: true, message: '请输入正确的内容', trigger: 'blur' }
        ]
      },
      users: [],
      sendnotice: false
    }
  },
  created() {
    this.getTicketUsers()
  },
  methods: {
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          putPayChannel(this.rowdata.id, this.rowdata).then(response => {
            if (this.sendnotice) {
              const messageForm = {
                action_user: this.rowdata.action_user,
                title: '【支付通道修改】',
                message: `修改人: ${this.create_user}\n处理人: ${this.rowdata.action_user}\n平台: ${this.rowdata.platform}     商户: ${this.rowdata.merchant}     通道: ${this.rowdata.type}`
              }
              postSendmessage(messageForm)
            }
            this.$emit('formdata', response.data)
            this.$refs[formName].resetFields()
          }).catch(error => {
            this.$message.error('检查修改项是否正确')
            console.log(error)
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getTicketUsers() {
      getUser().then(response => {
        this.users = response.data
      })
    }
  }
}
</script>
