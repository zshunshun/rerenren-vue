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
    <el-form-item label="电话" prop="telephone">
      <el-input v-model="dataForm.telephone" placeholder="电话"></el-input>
    </el-form-item>
    <el-form-item label="监护人姓名" prop="guardianName">
      <el-input v-model="dataForm.guardianName" placeholder="监护人姓名"></el-input>
    </el-form-item>
    <el-form-item label="监护人电话" prop="guardianTelephone">
      <el-input v-model="dataForm.guardianTelephone" placeholder="监护人电话"></el-input>
    </el-form-item>
    <el-form-item label="家庭住址" prop="address">
      <el-input v-model="dataForm.address" placeholder="家庭住址"></el-input>
    </el-form-item>
    <el-form-item label="相片" prop="pic">
      <el-input v-model="dataForm.pic" placeholder="相片"></el-input>
    </el-form-item>
<!--    <el-form-item label="钱包余额" prop="walletAmount">-->
<!--      <el-input v-model="dataForm.walletAmount" placeholder="钱包余额"></el-input>-->
<!--    </el-form-item>-->
<!--    <el-form-item label="健康档案" prop="healthRecord">-->
<!--      <el-input v-model="dataForm.healthRecord" placeholder="健康档案"></el-input>-->
<!--    </el-form-item>-->
<!--    <el-form-item label="操作人" prop="operationUserId">-->
<!--      <el-input v-model="dataForm.operationUserId" placeholder="操作人"></el-input>-->
<!--    </el-form-item>-->
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
        dataForm: {
          id: 0,
          name: '',
          sex: '',
          age: '',
          telephone: '',
          guardianName: '',
          guardianTelephone: '',
          address: '',
          pic: '',
          walletAmount: '',
          healthRecord: '',
          operationUserId: '',
          createTime: ''
        },
        dataRule: {
          name: [
            { required: true, message: '姓名不能为空', trigger: 'blur' }
          ],
          sex: [
            { required: true, message: '性别不能为空', trigger: 'blur' }
          ],
          age: [
            { required: true, message: '年龄不能为空', trigger: 'blur' }
          ],
          telephone: [
            { required: true, message: '电话不能为空', trigger: 'blur' }
          ],
          guardianName: [
            { required: true, message: '监护人姓名不能为空', trigger: 'blur' }
          ],
          guardianTelephone: [
            { required: true, message: '监护人电话不能为空', trigger: 'blur' }
          ],
          address: [
            { required: true, message: '家庭住址不能为空', trigger: 'blur' }
          ],
          pic: [
            { required: true, message: '相片不能为空', trigger: 'blur' }
          ],
          // walletAmount: [
          //   { required: true, message: '钱包余额不能为空', trigger: 'blur' }
          // ],
          // healthRecord: [
          //   { required: true, message: '健康档案不能为空', trigger: 'blur' }
          // ],
          // operationUserId: [
          //   { required: true, message: '操作人不能为空', trigger: 'blur' }
          // ],
          // createTime: [
          //   { required: true, message: '创建时间不能为空', trigger: 'blur' }
          // ]
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
              url: this.$http.adornUrl(`/generator/gafferinfo/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.gafferInfo.name
                this.dataForm.sex = data.gafferInfo.sex
                this.dataForm.age = data.gafferInfo.age
                this.dataForm.telephone = data.gafferInfo.telephone
                this.dataForm.guardianName = data.gafferInfo.guardianName
                this.dataForm.guardianTelephone = data.gafferInfo.guardianTelephone
                this.dataForm.address = data.gafferInfo.address
                this.dataForm.pic = data.gafferInfo.pic
                // this.dataForm.walletAmount = data.gafferInfo.walletAmount
                // this.dataForm.healthRecord = data.gafferInfo.healthRecord
                // this.dataForm.operationUserId = data.gafferInfo.operationUserId
                // this.dataForm.createTime = data.gafferInfo.createTime
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        // 判空
        if (this.dataForm.sex !== '男' && this.dataForm.sex !== '女') {
          this.$message.error('性别填写错误')
          return;
        }
        if (!/^[4-9][0-9]$|^1[0-9]{2}$/.test(this.dataForm.age)) {
          this.$message.error('年龄不符合要求')
          return;
        }
        if (!/^1[0-9]{10}$/.test(this.dataForm.telephone)) {
          this.$message.error('手机号码不符合要求')
          return;
        }
        if (!/^1[0-9]{10}$/.test(this.dataForm.guardianTelephone)) {
          this.$message.error('手机号码不符合要求')
          return;
        }
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/gafferinfo/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'sex': this.dataForm.sex,
                'age': this.dataForm.age,
                'telephone': this.dataForm.telephone,
                'guardianName': this.dataForm.guardianName,
                'guardianTelephone': this.dataForm.guardianTelephone,
                'address': this.dataForm.address,
                'pic': this.dataForm.pic,
                'healthRecord': this.dataForm.healthRecord,
                'operationUserId': this.dataForm.operationUserId,
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
