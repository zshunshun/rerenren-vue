<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="员工id" prop="empId">
      <el-input v-model="dataForm.empId" placeholder="员工id"></el-input>
    </el-form-item>
    <el-form-item label="本月基本工资" prop="amount">
      <el-input v-model="dataForm.amount" placeholder="本月基本工资"></el-input>
    </el-form-item>
    <el-form-item label="考评工资" prop="starAmount">
      <el-input v-model="dataForm.starAmount" placeholder="考评工资"></el-input>
    </el-form-item>
    <el-form-item label="考勤工资" prop="attendanceAmount">
      <el-input v-model="dataForm.attendanceAmount" placeholder="考勤工资"></el-input>
    </el-form-item>
    <el-form-item label="总工资" prop="totalAmount">
      <el-input v-model="dataForm.totalAmount" placeholder="总工资"></el-input>
    </el-form-item>
    <el-form-item label="操作人" prop="operationUserId">
      <el-input v-model="dataForm.operationUserId" placeholder="操作人"></el-input>
    </el-form-item>
    <el-form-item label="记录所属年月" prop="recordDate">
      <el-input v-model="dataForm.recordDate" placeholder="记录所属年月"></el-input>
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
          empId: '',
          amount: '',
          starAmount: '',
          attendanceAmount: '',
          totalAmount: '',
          operationUserId: '',
          recordDate: '',
          createTime: ''
        },
        dataRule: {
          empId: [
            { required: true, message: '员工id不能为空', trigger: 'blur' }
          ],
          amount: [
            { required: true, message: '本月基本工资不能为空', trigger: 'blur' }
          ],
          starAmount: [
            { required: true, message: '考评工资不能为空', trigger: 'blur' }
          ],
          attendanceAmount: [
            { required: true, message: '考勤工资不能为空', trigger: 'blur' }
          ],
          totalAmount: [
            { required: true, message: '总工资不能为空', trigger: 'blur' }
          ],
          operationUserId: [
            { required: true, message: '操作人不能为空', trigger: 'blur' }
          ],
          recordDate: [
            { required: true, message: '记录所属年月不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/empsalaryrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.empId = data.empSalaryRecord.empId
                this.dataForm.amount = data.empSalaryRecord.amount
                this.dataForm.starAmount = data.empSalaryRecord.starAmount
                this.dataForm.attendanceAmount = data.empSalaryRecord.attendanceAmount
                this.dataForm.totalAmount = data.empSalaryRecord.totalAmount
                this.dataForm.operationUserId = data.empSalaryRecord.operationUserId
                this.dataForm.recordDate = data.empSalaryRecord.recordDate
                this.dataForm.createTime = data.empSalaryRecord.createTime
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
              url: this.$http.adornUrl(`/generator/empsalaryrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'empId': this.dataForm.empId,
                'amount': this.dataForm.amount,
                'starAmount': this.dataForm.starAmount,
                'attendanceAmount': this.dataForm.attendanceAmount,
                'totalAmount': this.dataForm.totalAmount,
                'operationUserId': this.dataForm.operationUserId,
                'recordDate': this.dataForm.recordDate,
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
