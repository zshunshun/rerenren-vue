<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('generator:roomchange:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('generator:roomchange:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="gaffer.name"
        header-align="center"
        align="center"
        label="所属老人">
      </el-table-column>
      <el-table-column
        prop="beforeRoom.roomNo"
        header-align="center"
        align="center"
        label="变更前房间编号">
      </el-table-column>
      <el-table-column
        prop="beforeRoom.type"
        header-align="center"
        align="center"
        label="变更前房间类型">
      </el-table-column>
      <el-table-column
        prop="afterRoom.roomNo"
        header-align="center"
        align="center"
        label="变更后房间编号">
      </el-table-column>
      <el-table-column
        prop="afterRoom.type"
        header-align="center"
        align="center"
        label="变更前房间类型">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './roomchange-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      getGafferInfo() {
        let list = this.dataList;
        if (list && list.length > 0) {
          console.log('list:',list);
          let gafferIds = [];
          list.forEach(item => {
            gafferIds.push(item.theAgedId);
          })
          // 获取老人信息
          this.$http({
            url: this.$http.adornUrl('/generator/gafferinfo/list'),
            method: 'get',
            params: this.$http.adornParams({
              'gafferIdList': gafferIds.join(',')
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              console.log('gafferInfo',data.page.list);
              let dataList = data.page.list;
              if (dataList && dataList.length > 0) {
                this.dataList.forEach(item => {
                  dataList.forEach(gaffer => {
                    if (item.theAgedId === gaffer.id) {
                      this.$set(item,'gaffer',gaffer);
                      return;
                    }
                  })
                })
              }
            }
          })
        }
      },
      getBeforeRoomInfo() {
        let list = this.dataList;
        if (list && list.length > 0) {
          console.log('list:',list);
          let roomIds = [];
          list.forEach(item => {
            roomIds.push(item.beforeRoomId);
          })
          // 获取老人信息
          this.$http({
            url: this.$http.adornUrl('/generator/roominfo/list'),
            method: 'get',
            params: this.$http.adornParams({
              'roomIdList': roomIds.join(',')
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              console.log('roomIdList',data.page.list);
              let dataList = data.page.list;
              if (dataList && dataList.length > 0) {
                this.dataList.forEach(item => {
                  dataList.forEach(root => {
                    if (item.beforeRoomId === root.id) {
                      this.$set(item,'beforeRoom',root);
                      return;
                    }
                  })
                })
              }
            }
          })
        }
      },
      getAfterRoomInfo() {
        let list = this.dataList;
        if (list && list.length > 0) {
          console.log('list:',list);
          let roomIds = [];
          list.forEach(item => {
            roomIds.push(item.nowRoomId);
          })
          // 获取老人信息
          this.$http({
            url: this.$http.adornUrl('/generator/roominfo/list'),
            method: 'get',
            params: this.$http.adornParams({
              'roomIdList': roomIds.join(',')
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              console.log('roomIdList',data.page.list);
              let dataList = data.page.list;
              if (dataList && dataList.length > 0) {
                this.dataList.forEach(item => {
                  dataList.forEach(root => {
                    if (item.nowRoomId === root.id) {
                      this.$set(item,'afterRoom',root);
                      return;
                    }
                  })
                })
              }
            }
          })
        }
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/roomchange/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
            this.getGafferInfo();
            this.getBeforeRoomInfo();
            this.getAfterRoomInfo();
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/roomchange/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
