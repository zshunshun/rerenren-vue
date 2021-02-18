<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="老人账户id" prop="gafferId">
      <el-input v-model="dataForm.gafferId" placeholder="老人账户id"></el-input>
    </el-form-item>
    <el-form-item label="消费金额" prop="amount">
      <el-input v-model="dataForm.amount" placeholder="消费金额"></el-input>
    </el-form-item>
    <el-form-item label="消费详情" prop="detail">
      <el-input v-model="dataForm.detail" placeholder="消费详情"></el-input>
    </el-form-item>
    <el-form-item label="审核状态：0-提交 1-审核通过 2-审核拒绝" prop="status">
      <el-input v-model="dataForm.status" placeholder="审核状态：0-提交 1-审核通过 2-审核拒绝"></el-input>
    </el-form-item>
    <el-form-item label="提交医护id" prop="operationId">
      <el-input v-model="dataForm.operationId" placeholder="提交医护id"></el-input>
    </el-form-item>
    <el-form-item label="创建时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
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
          id: 0,
          gafferId: '',
          amount: '',
          detail: '',
          status: '',
          operationId: '',
          createTime: ''
        },
        dataRule: {
          gafferId: [
            { required: true, message: '老人账户id不能为空', trigger: 'blur' }
          ],
          amount: [
            { required: true, message: '消费金额不能为空', trigger: 'blur' }
          ],
          detail: [
            { required: true, message: '消费详情不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '审核状态：0-提交 1-审核通过 2-审核拒绝不能为空', trigger: 'blur' }
          ],
          operationId: [
            { required: true, message: '提交医护id不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/gafferpayment/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.gafferId = data.gafferPayment.gafferId
                this.dataForm.amount = data.gafferPayment.amount
                this.dataForm.detail = data.gafferPayment.detail
                this.dataForm.status = data.gafferPayment.status
                this.dataForm.operationId = data.gafferPayment.operationId
                this.dataForm.createTime = data.gafferPayment.createTime
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
              url: this.$http.adornUrl(`/generator/gafferpayment/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'gafferId': this.dataForm.gafferId,
                'amount': this.dataForm.amount,
                'detail': this.dataForm.detail,
                'status': this.dataForm.status,
                'operationId': this.dataForm.operationId,
                'createTime': this.dataForm.createTime
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
