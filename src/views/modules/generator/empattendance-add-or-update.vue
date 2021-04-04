<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="员工" prop="empId">
      <!-- <el-input v-model="dataForm.empId" placeholder="员工id"></el-input> -->
      <el-select
      v-model="dataForm.empId"
      filterable
      remote
      reserve-keyword
      placeholder="请输入姓名或编号"
      :remote-method="remoteSearchEmp"
      :loading="loading">
      <el-option
        v-for="item in empList"
        :key="item.id"
        :label="item.name+':'+item.department+':'+item.job"
        :value="item.id">
      </el-option>
  </el-select>
    </el-form-item>
    <el-form-item label="考勤月份" prop="attendanceDate">
      <!-- <el-input v-model="dataForm.attendanceDate" placeholder="考勤所属年月"></el-input> -->
      <!-- <span class="demonstration">月</span> -->
      <el-date-picker
        v-model="dataForm.attendanceDate"
        type="month"
        placeholder="选择月">
      </el-date-picker>
    </el-form-item>
    <el-form-item label="应到天数" prop="shouldDay">
      <!-- <el-input v-model="dataForm.shouldDay" placeholder="应到天数"></el-input> -->
      <el-input-number v-model="dataForm.shouldDay" :min="0" :max="31" label="应到天数"></el-input-number>
    </el-form-item>
    <el-form-item label="实到天数" prop="actualDay">
      <!-- <el-input v-model="dataForm.actualDay" placeholder="实到天数"></el-input> -->
      <el-input-number v-model="dataForm.actualDay" :min="0" :max="31" label="实到天数"></el-input-number>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { formatDate } from '@/utils/dateUtils';
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
        },
        empList: [],
        loading: false
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
                this.initEmpList(data.empAttendance.empId)
                this.dataForm.empId = data.empAttendance.empId
                this.dataForm.attendanceDate = data.empAttendance.attendanceDate
                this.dataForm.shouldDay = data.empAttendance.shouldDay
                this.dataForm.actualDay = data.empAttendance.actualDay
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            let attendanceDate = this.dataForm.attendanceDate;
            this.$http({
              url: this.$http.adornUrl(`/generator/empattendance/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'empId': this.dataForm.empId,
                'attendanceDate': (typeof(attendanceDate) != 'string') ? formatDate(attendanceDate,"yyyy-MM") : attendanceDate,
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
      },
      initEmpList(empId) {
        this.$http({
          url: this.$http.adornUrl('/generator/empinfo/info/' + empId),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            console.log(data.empInfo);
            this.empList = [data.empInfo];
          } else {
            this.empList = []
          }
          // this.loading = false
        })
      },
      // 搜索
      remoteSearchEmp(keyword) {
        if(keyword != '') {
          this.loading = true;
          this.$http({
            url: this.$http.adornUrl('/generator/empinfo/searchForSalary'),
            method: 'get',
            params: this.$http.adornParams({
              'keyword': keyword
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              console.log(data.empList);
              this.empList = data.empList;
            } else {
              this.empList = []
            }
            this.loading = false
          })
        }
      }
    }
  }
</script>
