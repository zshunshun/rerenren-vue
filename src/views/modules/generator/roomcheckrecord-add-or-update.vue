<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
<!--    <el-form-item label="老人账户id" prop="gafferId">-->
<!--      <el-input v-model="dataForm.gafferId" placeholder="老人账户id"></el-input>-->
<!--    </el-form-item>-->
      <el-form-item label="老人账户" prop="gafferId">
        <el-select
          v-model="dataForm.gafferId"
          filterable
          remote
          reserve-keyword
          @change="changeSelect"
          placeholder="请输入关键词"
          :remote-method="remoteSearchGaffer"
          :loading="loading">
          <el-option
            v-for="item in gafferList"
            :key="item.id"
            :label="item.name+':'+item.telephone"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
<!--    <el-form-item label="查房人id" prop="operationId">-->
<!--      <el-input v-model="dataForm.operationId" placeholder="查房人id"></el-input>-->
<!--    </el-form-item>-->
    <el-form-item label="查房内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="查房内容"></el-input>
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
  export default {
    data () {
      return {
        visible: false,
        gafferList: '',
        dataForm: {
          id: 0,
          gafferId: '',
          operationId: '',
          content: '',
          createTime: ''
        },
        dataRule: {
          gafferId: [
            { required: true, message: '老人账户id不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '查房内容不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/roomcheckrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.gafferId = data.roomCheckRecord.gafferId
                this.dataForm.content = data.roomCheckRecord.content
              }
            })
          }
        })
      },
      getEmpData(empIdList,attendanceData) {
        let empIds = empIdList.map(item => item).join(',');
        this.$http({
          url: this.$http.adornUrl('/generator/empinfo/listByIds?empIdList=' + empIds),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0 && data.empList && data.empList.length > 0) {
            attendanceData.page.list.forEach(attendance => {
              data.empList.forEach(emp => {
                if (attendance.empId === emp.id) {
                  attendance.empName = emp.name
                  return;
                }
              })
            })
          } else {
            this.$notify.error({
              title: '错误',
              message: '获取员工数据失败'
            });
          }
          this.dataList = attendanceData.page.list
          this.totalPage = attendanceData.page.totalCount
        })
      },
      remoteSearchGaffer(keyword) {
        if(keyword != '') {
          this.loading = true;
          this.$http({
            url: this.$http.adornUrl('/generator/gafferinfo/search'),
            method: 'get',
            params: this.$http.adornParams({
              'keyword': keyword
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.gafferList = data.gafferInfoList;
            } else {
              this.gafferList = []
            }
            this.loading = false
          })
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/roomcheckrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'gafferId': this.dataForm.gafferId,
                'operationId': this.dataForm.operationId,
                'content': this.dataForm.content,
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
