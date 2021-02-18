<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="员工id" prop="empId">
      <el-input v-model="dataForm.empId" placeholder="员工id"></el-input>
    </el-form-item>
    <el-form-item label="考勤所属年月" prop="attendanceDate">
      <el-input v-model="dataForm.attendanceDate" placeholder="考勤所属年月"></el-input>
    </el-form-item>
    <el-form-item label="应到天数" prop="shouldDay">
      <el-input v-model="dataForm.shouldDay" placeholder="应到天数"></el-input>
    </el-form-item>
    <el-form-item label="实到天数" prop="actualDay">
      <el-input v-model="dataForm.actualDay" placeholder="实到天数"></el-input>
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
          attendanceDate: '',
          shouldDay: '',
          actualDay: '',
          createTime: ''
        },
        dataRule: {
          empId: [
            { required: true, message: '员工id不能为空', trigger: 'blur' }
          ],
          attendanceDate: [
            { required: true, message: '考勤所属年月不能为空', trigger: 'blur' }
          ],
          shouldDay: [
            { required: true, message: '应到天数不能为空', trigger: 'blur' }
          ],
          actualDay: [
            { required: true, message: '实到天数不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/empattendance/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.empId = data.empAttendance.empId
                this.dataForm.attendanceDate = data.empAttendance.attendanceDate
                this.dataForm.shouldDay = data.empAttendance.shouldDay
                this.dataForm.actualDay = data.empAttendance.actualDay
                this.dataForm.createTime = data.empAttendance.createTime
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
              url: this.$http.adornUrl(`/generator/empattendance/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'empId': this.dataForm.empId,
                'attendanceDate': this.dataForm.attendanceDate,
                'shouldDay': this.dataForm.shouldDay,
                'actualDay': this.dataForm.actualDay,
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
