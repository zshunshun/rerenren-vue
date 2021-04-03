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
    <el-form-item label="缴费金额" prop="amount">
      <el-input v-model="dataForm.amount" placeholder="缴费金额"></el-input>
    </el-form-item>
      <el-form-item label="所属月份" prop="attendanceDate">
        <el-date-picker
          v-model="dataForm.paymentDate"
          type="month"
          placeholder="选择月">
        </el-date-picker>
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
          gafferId: '',
          amount: '',
          paymentDate: '',
          createTime: ''
        },
        dataRule: {
          gafferId: [
            { required: true, message: '老人账户id不能为空', trigger: 'blur' }
          ],
          amount: [
            { required: true, message: '缴费金额不能为空', trigger: 'blur' }
          ],
          paymentDate: [
            { required: true, message: '缴费所属年月不能为空', trigger: 'blur' }
          ]
        },
        loading: false,
        gafferList: [],
        selectGaffer: {},
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
              url: this.$http.adornUrl(`/generator/gafferpaymentrecord/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.gafferId = data.gafferPaymentRecord.gafferId
                this.dataForm.amount = data.gafferPaymentRecord.amount
                this.dataForm.paymentDate = data.gafferPaymentRecord.paymentDate
                // this.dataForm.createTime = data.gafferPaymentRecord.createTime
                this.initGafferList(data.gafferPaymentRecord.gafferId);
              }
            })
          }
        })
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
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/gafferpaymentrecord/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'gafferId': this.dataForm.gafferId,
                'amount': this.dataForm.amount,
                'paymentDate': this.dataForm.paymentDate,
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
    }
  }
</script>
