<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
<!--    <el-form-item label="所属老人id" prop="theAgedId">-->
<!--      <el-input v-model="dataForm.theAgedId" placeholder="所属老人id"></el-input>-->
<!--    </el-form-item>-->
    <el-form-item label="老人账户" prop="theAgedId">
      <el-select
        v-model="dataForm.theAgedId"
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
<!--    <el-form-item label="变更前房间号" prop="beforeRoomId">-->
<!--      <el-input v-model="dataForm.beforeRoomId" placeholder="变更前房间id"></el-input>-->
<!--    </el-form-item>-->
      <el-form-item label="变更前房间" prop="beforeRoomId">
        <el-select
          v-model="dataForm.beforeRoomId"
          filterable
          remote
          reserve-keyword
          @change="changeSelectRoom"
          placeholder="请输入关键词"
          :remote-method="remoteSearchRoom"
          :loading="loading">
          <el-option
            v-for="item in beforeRoomList"
            :key="item.id"
            :label="item.roomNo+':'+item.type"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
<!--    <el-form-item label="变更后房间号" prop="nowRoomId">-->
<!--      <el-input v-model="dataForm.nowRoomId" placeholder="变更后房间id"></el-input>-->
<!--    </el-form-item>-->
      <el-form-item label="变更后房间" prop="nowRoomId">
        <el-select
          v-model="dataForm.nowRoomId"
          filterable
          remote
          reserve-keyword
          @change="changeSelectRoom"
          placeholder="请输入关键词"
          :remote-method="remoteSearchRoom2"
          :loading="loading">
          <el-option
            v-for="item in afterRoomList"
            :key="item.id"
            :label="item.roomNo+':'+item.type"
            :value="item.id">
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
        selectGaffer: '',
        loading: false,
        gafferList: [],
        beforeRoomList: '',
        afterRoomList: '',
        selectBeforeRoomId: [],
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
              url: this.$http.adornUrl(`/generator/roomchange/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.theAgedId = data.roomChange.theAgedId
                this.dataForm.beforeRoomId = data.roomChange.beforeRoomId
                this.dataForm.nowRoomId = data.roomChange.nowRoomId
                this.initGafferList(data.roomChange.theAgedId);
                // this.dataForm.createTime = data.roomChange.createTime
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
        let gafferId = this.dataForm.theAgedId;
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
      changeSelectRoom(e) {
        let beforeRoomId = this.dataForm.beforeRoomId;
        console.log(beforeRoomId);
        if (this.beforeRoomList && this.beforeRoomList.length > 0) {
          this.beforeRoomList.forEach(item => {
            if (item.id === beforeRoomId) {
              this.selectBeforeRoomId = item;
            }
          })
        }
        this.showWalletAmount();
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
      remoteSearchRoom(keyword) {
        if(keyword != '') {
          this.loading = true;
          this.$http({
            url: this.$http.adornUrl('/generator/roominfo/search'),
            method: 'get',
            params: this.$http.adornParams({
              'keyword': keyword
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.beforeRoomList = data.roomInfoList;
            } else {
              this.beforeRoomList = []
            }
            this.loading = false
          })
        }
      },
      remoteSearchRoom2(keyword) {
        if(keyword != '') {
          this.loading = true;
          this.$http({
            url: this.$http.adornUrl('/generator/roominfo/search'),
            method: 'get',
            params: this.$http.adornParams({
              'keyword': keyword
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.afterRoomList = data.roomInfoList;
            } else {
              this.afterRoomList = []
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
