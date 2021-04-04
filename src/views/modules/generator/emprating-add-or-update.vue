<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
    <el-form-item label="员工" prop="empId">
<!--      <el-input v-model="dataForm.empId" placeholder="员工id"></el-input>-->
      <el-select
        v-model="dataForm.empId"
        filterable
        remote
        reserve-keyword
        placeholder="请输入关键词"
        :remote-method="remoteSearchEmp"
        :loading="loading_emp">
        <el-option
          v-for="item in empList"
          :key="item.id"
          :label="item.name+':'+item.department+':'+item.job"
          :value="item.id">
        </el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="评价人" prop="userId">
<!--      <el-input v-model="dataForm.userId" placeholder="评价人id"></el-input>-->
      <el-select
        v-model="dataForm.userId"
        filterable
        remote
        reserve-keyword
        placeholder="请输入关键词"
        :remote-method="remoteSearchUser"
        :loading="loading_user">
        <el-option
          v-for="item in userList"
          :key="item.id"
          :label="item.username+':'+item.mobile"
          :value="item.userId">
        </el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="星级" prop="star">
<!--      <el-input v-model="dataForm.star" placeholder="星级"></el-input>-->
      <el-rate
        v-model="dataForm.star"
        show-text>
      </el-rate>
    </el-form-item>
    <el-form-item label="评级所属年月" prop="starDate">
<!--      <el-input v-model="dataForm.starDate" placeholder="评级所属年月"></el-input>-->
      <el-date-picker
        v-model="dataForm.starDate"
        type="month"
        placeholder="选择月">
      </el-date-picker>
    </el-form-item>
<!--    <el-form-item label="创建时间" prop="createTime">-->
<!--      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>-->
<!--    </el-form-item>-->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { formatDate } from '@/utils/dateUtils'
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          empId: '',
          userId: '',
          star: '',
          starDate: '',
          createTime: ''
        },
        dataRule: {
          empId: [
            { required: true, message: '员工不能为空', trigger: 'blur' }
          ],
          userId: [
            { required: true, message: '评价人不能为空', trigger: 'blur' }
          ],
          star: [
            { required: true, message: '星级不能为空', trigger: 'blur' }
          ],
          starDate: [
            { required: true, message: '评级所属年月不能为空', trigger: 'blur' }
          ],
        },
        empList: [],
        userList: [],
        loading_emp: false,
        loading_user: false
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
              url: this.$http.adornUrl(`/generator/emprating/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.initEmpList(data.empRating.empId)
                this.initUserList(data.empRating.userId)
                this.dataForm.empId = data.empRating.empId
                this.dataForm.userId = data.empRating.userId
                this.dataForm.star = data.empRating.star
                this.dataForm.starDate = data.empRating.starDate //new Date(data.empRating.starDate.split('-')[0],data.empRating.starDate.split('-')[1])
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            let starDate = this.dataForm.starDate;
            this.$http({
              url: this.$http.adornUrl(`/generator/emprating/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'empId': this.dataForm.empId,
                'userId': this.dataForm.userId,
                'star': this.dataForm.star,
                'starDate': (typeof(starDate) != 'string') ? formatDate(starDate,'yyyy-MM') : starDate,
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
            this.empList = [data.empInfo];
          } else {
            this.empList = []
          }
          // this.loading = false
        })
      },
      initUserList(userId) {
        this.$http({
          url: this.$http.adornUrl('/sys/user/userInfo/' + userId),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            console.log(data.user);
            this.userList = [data.user];
          } else {
            this.userList = []
          }
        })
        console.log(this.userList);
      },
      remoteSearchEmp(keyword) {
        if(keyword != '') {
          this.loading_emp = true;
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
            this.loading_emp = false
          })
        }
      },
      remoteSearchUser(keyword) {
        if(keyword != '') {
          this.loading_user = true;
          this.$http({
            url: this.$http.adornUrl('/sys/user/search'),
            method: 'get',
            params: this.$http.adornParams({
              'keyword': keyword
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.userList = data.userList;
            } else {
              this.userList = []
            }
            this.loading_user = false
          })
        }
      }
    }
  }
</script>
