<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="所属老人id" prop="theAgedId">
      <el-input v-model="dataForm.theAgedId" placeholder="所属老人id"></el-input>
    </el-form-item>
    <el-form-item label="变更前房间id" prop="beforeRoomId">
      <el-input v-model="dataForm.beforeRoomId" placeholder="变更前房间id"></el-input>
    </el-form-item>
    <el-form-item label="变更后房间id" prop="nowRoomId">
      <el-input v-model="dataForm.nowRoomId" placeholder="变更后房间id"></el-input>
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
          theAgedId: '',
          beforeRoomId: '',
          nowRoomId: '',
          createTime: ''
        },
        dataRule: {
          theAgedId: [
            { required: true, message: '所属老人id不能为空', trigger: 'blur' }
          ],
          beforeRoomId: [
            { required: true, message: '变更前房间id不能为空', trigger: 'blur' }
          ],
          nowRoomId: [
            { required: true, message: '变更后房间id不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/roomchange/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.theAgedId = data.roomChange.theAgedId
                this.dataForm.beforeRoomId = data.roomChange.beforeRoomId
                this.dataForm.nowRoomId = data.roomChange.nowRoomId
                this.dataForm.createTime = data.roomChange.createTime
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
              url: this.$http.adornUrl(`/generator/roomchange/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'theAgedId': this.dataForm.theAgedId,
                'beforeRoomId': this.dataForm.beforeRoomId,
                'nowRoomId': this.dataForm.nowRoomId,
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
