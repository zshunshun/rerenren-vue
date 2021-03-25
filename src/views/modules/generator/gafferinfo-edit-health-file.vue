<template>
  <el-dialog title="健康档案" :visible.sync="dialogHealthVisible" :close-on-click-modal="false">
      <p>姓名：{{dataForm.name}}</p>

      <script :id="ueId" class="ueditor-box" type="text/plain" style="width: 100%; height: 260px;"></script>

    <span slot="footer" class="dialog-footer">
      <el-button @click="dialogHealthVisible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">保存</el-button>
    </span>
  </el-dialog>
</template>

<script>
import ueditor from 'ueditor'
export default {
  data () {
    return {
      dataForm: {
        id: 0,
        name: '',
        healthRecord: ''
      },
      dialogHealthVisible: true,
      ue: null,
      ueId: `J_ueditorBox_${new Date().getTime()}`,
      ueContent: '',
      dialogVisible: false
    }
  },
  methods: {
    getContent () {
      this.dialogVisible = true
      this.ue.ready(() => {
        this.ueContent = this.ue.getContent()
      })
    },
    init (id) {
      this.dialogHealthVisible = true;
      this.ue = ueditor.getEditor(this.ueId, {
        // serverUrl: '', // 服务器统一请求接口路径
        // zIndex: 3000
      })
      this.dataForm.id = id || 0
      this.$nextTick(() => {
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/generator/gafferinfo/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.gafferInfo.name
              this.dataForm.healthRecord = data.gafferInfo.healthRecord
              this.ue.ready(() => {
                this.ue.setContent(data.gafferInfo.healthRecord);
              })
            }
          })
        }
      })
    },
    dataFormSubmit () {
      this.ue.ready(() => {
        this.dataForm.healthRecord = this.ue.getContent()
      })
      // 判空
      if (!this.dataForm.healthRecord) {
        this.$message.error('档案不能为空')
        return;
      }
      if (!this.dataForm.id) {
        this.$message.error('未选择有效用户，请重新选择')
        return;
      }
      this.$http({
        url: this.$http.adornUrl(`/generator/gafferinfo/update`),
        method: 'post',
        data: this.$http.adornData({
          'id': this.dataForm.id || undefined,
          'healthRecord': this.dataForm.healthRecord,
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
          this.dialogHealthVisible = false;
        } else {
          this.$message.error(data.msg)
        }
      })
    }
  }
}
</script>

