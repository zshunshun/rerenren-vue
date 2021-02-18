<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="房间号" prop="roomNo">
      <el-input v-model="dataForm.roomNo" placeholder="房间号"></el-input>
    </el-form-item>
    <el-form-item label="房间类型" prop="type">
      <el-input v-model="dataForm.type" placeholder="房间类型"></el-input>
    </el-form-item>
    <el-form-item label="房间价格（按天）" prop="price">
      <el-input v-model="dataForm.price" placeholder="房间价格（按天）"></el-input>
    </el-form-item>
    <el-form-item label="当前状态：0-空置 1-占用" prop="status">
      <el-input v-model="dataForm.status" placeholder="当前状态：0-空置 1-占用"></el-input>
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
          roomNo: '',
          type: '',
          price: '',
          status: ''
        },
        dataRule: {
          roomNo: [
            { required: true, message: '房间号不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '房间类型不能为空', trigger: 'blur' }
          ],
          price: [
            { required: true, message: '房间价格（按天）不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '当前状态：0-空置 1-占用不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/roominfo/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.roomNo = data.roomInfo.roomNo
                this.dataForm.type = data.roomInfo.type
                this.dataForm.price = data.roomInfo.price
                this.dataForm.status = data.roomInfo.status
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
              url: this.$http.adornUrl(`/generator/roominfo/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'roomNo': this.dataForm.roomNo,
                'type': this.dataForm.type,
                'price': this.dataForm.price,
                'status': this.dataForm.status
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
