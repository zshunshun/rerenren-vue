<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="姓名" prop="name">
      <el-input v-model="dataForm.name" placeholder="姓名"></el-input>
    </el-form-item>
    <el-form-item label="性别" prop="sex">
      <el-input v-model="dataForm.sex" placeholder="性别"></el-input>
    </el-form-item>
    <el-form-item label="年龄" prop="age">
      <el-input v-model="dataForm.age" placeholder="年龄"></el-input>
    </el-form-item>
    <el-form-item label="科室" prop="department">
      <el-input v-model="dataForm.department" placeholder="科室"></el-input>
    </el-form-item>
    <el-form-item label="职称" prop="job">
      <el-input v-model="dataForm.job" placeholder="职称"></el-input>
    </el-form-item>
    <el-form-item label="描述" prop="description">
      <el-input v-model="dataForm.description" placeholder="描述"></el-input>
    </el-form-item>
    <el-form-item label="相片" prop="pic">
      <el-input v-model="dataForm.pic" placeholder="相片"></el-input>
    </el-form-item>
    <el-form-item label="基本工资" prop="basicSalary">
      <el-input v-model="dataForm.basicSalary" placeholder="基本工资"></el-input>
    </el-form-item>
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
          no: '',
          name: '',
          sex: '',
          age: '',
          department: '',
          job: '',
          description: '',
          pic: '',
          basicSalary: '',
          createTime: ''
        },
        dataRule: {
          no: [
            { required: true, message: '编号不能为空', trigger: 'blur' }
          ],
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          sex: [
            { required: true, message: '性别不能为空', trigger: 'blur' }
          ],
          age: [
            { required: true, message: '年龄不能为空', trigger: 'blur' }
          ],
          department: [
            { required: true, message: '科室不能为空', trigger: 'blur' }
          ],
          job: [
            { required: true, message: '职称不能为空', trigger: 'blur' }
          ],
          description: [
            { required: true, message: '描述不能为空', trigger: 'blur' }
          ],
          pic: [
            { required: true, message: '相片不能为空', trigger: 'blur' }
          ],
          basicSalary: [
            { required: true, message: '基本工资不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/empinfo/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.no = data.empInfo.no
                this.dataForm.name = data.empInfo.name
                this.dataForm.sex = data.empInfo.sex
                this.dataForm.age = data.empInfo.age
                this.dataForm.department = data.empInfo.department
                this.dataForm.job = data.empInfo.job
                this.dataForm.description = data.empInfo.description
                this.dataForm.pic = data.empInfo.pic
                this.dataForm.basicSalary = data.empInfo.basicSalary
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        console.log(this.dataForm.createTime);
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/empinfo/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'no': this.dataForm.no,
                'name': this.dataForm.name,
                'sex': this.dataForm.sex,
                'age': this.dataForm.age,
                'department': this.dataForm.department,
                'job': this.dataForm.job,
                'description': this.dataForm.description,
                'pic': this.dataForm.pic,
                'basicSalary': this.dataForm.basicSalary
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
