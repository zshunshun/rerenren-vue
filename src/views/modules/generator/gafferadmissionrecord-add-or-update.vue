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
    <el-form-item label="账户余额" prop="wallet" v-if="isShowWalletAmount">
      {{selectGaffer.walletAmount}}
      <el-button type="primary" @click="rebackAmount">已退余额</el-button>
    </el-form-item>
    <el-form-item label="出/入院" prop="type">
      <el-select v-model="dataForm.type" @change="showWalletAmount" placeholder="出/入院">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
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
        options: [{
          value: '出院',
          label: '出院'
          },
          {
            value: '入院',
            label: '入院'
          }],
        dataForm: {
          id: 0,
          gafferId: '',
          type: '',
          createTime: ''
        },
        dataRule: {
          gafferId: [
            { required: true, message: '老人账户id不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '出院/入院不能为空', trigger: 'blur' }
          ]
        },
        gafferList: [],
        selectGaffer: {},
        loading: false,
        isShowWalletAmount: false
      }
    },
    methods: {
      rebackAmount() {

      },
      /**
       * 计算是否显示退余额
       */
      showWalletAmount() {
        this.isShowWalletAmount = this.selectGaffer.walletAmount > 0 && this.dataForm.type === '出院';
      },
      changeSelect(e) {
        let gafferId = this.dataForm.gafferId;
        console.log(gafferId);
        if (this.gafferList && this.gafferList.length > 0) {
          this.gafferList.forEach(item => {
            if (item.id === gafferId) {
              this.selectGaffer = item;
            }
          })
        }
        this.showWalletAmount();
      },
      initGafferList(gafferId) {
        this.$http({
          url: this.$http.adornUrl('/generator/gafferinfo/info/' + gafferId),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            console.log(data.gafferInfo);
            this.gafferList = [data.gafferInfo];
            this.selectGaffer = data.gafferInfo
          } else {
            this.gafferList = []
          }
        })
        console.log(this.userList);
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
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/gafferadmissionrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.gafferId = data.gafferAdmissionRecord.gafferId
                this.dataForm.type = data.gafferAdmissionRecord.type
                this.dataForm.createTime = data.gafferAdmissionRecord.createTime
                this.initGafferList(data.gafferAdmissionRecord.gafferId);
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
              url: this.$http.adornUrl(`/generator/gafferadmissionrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'gafferId': this.dataForm.gafferId,
                'type': this.dataForm.type,
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
