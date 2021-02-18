<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="员工id" prop="empId">
      <el-input v-model="dataForm.empId" placeholder="员工id"></el-input>
    </el-form-item>
    <el-form-item label="评价人id" prop="userId">
      <el-input v-model="dataForm.userId" placeholder="评价人id"></el-input>
    </el-form-item>
    <el-form-item label="星级" prop="star">
      <el-input v-model="dataForm.star" placeholder="星级"></el-input>
    </el-form-item>
    <el-form-item label="评级所属年月" prop="starDate">
      <el-input v-model="dataForm.starDate" placeholder="评级所属年月"></el-input>
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
          userId: '',
          star: '',
          starDate: '',
          createTime: ''
        },
        dataRule: {
          empId: [
            { required: true, message: '员工id不能为空', trigger: 'blur' }
          ],
          userId: [
            { required: true, message: '评价人id不能为空', trigger: 'blur' }
          ],
          star: [
            { required: true, message: '星级不能为空', trigger: 'blur' }
          ],
          starDate: [
            { required: true, message: '评级所属年月不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/emprating/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.empId = data.empRating.empId
                this.dataForm.userId = data.empRating.userId
                this.dataForm.star = data.empRating.star
                this.dataForm.starDate = data.empRating.starDate
                this.dataForm.createTime = data.empRating.createTime
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
              url: this.$http.adornUrl(`/generator/emprating/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'empId': this.dataForm.empId,
                'userId': this.dataForm.userId,
                'star': this.dataForm.star,
                'starDate': this.dataForm.starDate,
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
